# Guida completa alle DS Primitives

## Panoramica
Le DS (Design System) primitives di Primer sono il catalogo ufficiale di valori per colori, spaziatura, tipografia e motion utilizzati da tutti i prodotti GitHub. Il pacchetto `@primer/primitives` distribuisce questi valori già normalizzati, pronti per essere consumati tramite CSS, JavaScript/TypeScript o JSON.

- Il pacchetto si installa via npm e vive su `@primer/primitives`.
- I dati compilati vengono pubblicati in `dist/`, mentre i sorgenti risiedono in `src/tokens/`.
- L'esperienza completa (storybook, documentazione, strumenti di validazione) è pensata per garantire coerenza cromatica e tipografica in tutte le superfici prodotto.

## Livelli di token
L'architettura è stratificata per rendere chiaro il passaggio da valori puri a semantici:

| Livello | Percorso | Scopo |
| --- | --- | --- |
| Base | `src/tokens/base/*` | Valori grezzi (scale cromatiche, space scale, tipografia, motion). |
| Funzionale | `src/tokens/functional/*` | Token semantici collegati a usi specifici (es. `bgColor`, `border`, `shadow`, `control`). |
| Componenti | `src/tokens/component/*` | Token ottimizzati per componenti GitHub (button, overlay, menu, ecc.). |
| Fallback/Removed | `src/tokens/fallback` e `src/tokens/removed` | Supporto a versioni legacy o deprecazioni guidate. |

Questa separazione permette di mantenere stabili i layer di base mentre i layer funzionali e componenti possono evolvere per soddisfare requisiti d'interfaccia senza rompere i consumer.

## Anatomia di un token
Ogni token segue la specifica W3C Design Tokens (`$value`, `$type`, `$extensions`, ecc.) e può includere override per i diversi temi. Ad esempio, il token `bgColor.default` definisce il valore light, i tipi e gli override per i temi dark o high-contrast:

```2:70:src/tokens/functional/color/bgColor.json5
{
  bgColor: {
    default: {
      $value: '{base.color.neutral.0}',
      $type: 'color',
      $extensions: {
        'org.primer.figma': {
          collection: 'mode',
          scopes: ['bgColor', 'borderColor'],
          group: 'semantic',
          codeSyntax: {
            web: 'var(--bgColor-default) /* utility class: .color-bg-default */'
          }
        },
        'org.primer.overrides': {
          dark: '{base.color.neutral.1}',
          'dark-dimmed': '{base.color.neutral.3}',
          ...
        }
      }
    },
    ...
  }
}
```

L'oggetto `$extensions` è usato per:

- Metadati Figma (`collection`, `mode`, `scopes`, snippet di codice).
- Override automatici per i temi (`org.primer.overrides`).
- Qualsiasi altro metadato interno (es. gruppi semantici, documentazione automatica).

### Overrides e @-hack
- Gli override risiedono in `src/tokens/functional/color/<mode>/overrides` e vengono attivati dichiarandoli in `scripts/themes.config.ts`. In questo modo è sufficiente specificare soltanto la differenza rispetto al tema principale.
- Il cosiddetto "@-hack" permette di avere un valore di default e delle varianti (es. `bgColor.accent.@` come default e `bgColor.accent.muted` come sottovalore) senza violare la validazione dei nomi dei token.

## Flusso di build e validazione
Tutte le operazioni di generazione sono orchestrate da `package.json`:

```25:46:package.json
"scripts": {
  "build": "npm run clean && npm run build:tokens && npm run build:fallbacks && npm run build:figma && npm run build:config",
  "build:tokens": "tsx ./scripts/buildTokens.ts",
  "build:fallbacks": "tsx ./scripts/buildFallbacks.ts",
  "build:figma": "tsx scripts/buildFigma.ts",
  "build:config": "tsc -p build.tsconfig.jsonc && tsx ./scripts/copyDir.ts src/types dist/build/types",
  "clean": "rm -rf dist",
  "lint": "eslint ... && npm run lint:tokens",
  "lint:tokens": "tsx ./scripts/validateTokenJson.ts",
  ...
}
```

Workflow consigliato:

1. **Modifica** i file in `src/tokens/` (usa JSON5 per commenti e trailing comma).
2. **Valida**: `npm run lint:tokens` garantisce naming corretto e rispetto dello schema.
3. **Compila**: `npm run build` popola `dist/` con CSS, JSON, config e asset Figma.
4. **Verifica**: apri Storybook (`npm run start:storybook`) per controllare l'effetto visivo.

## Come consumare i token
### Installazione
```sh
npm install --save @primer/primitives
```
Una volta installato, puoi importare solo i segmenti necessari per limitare il peso.

### CSS (consumo diretto)
Tutti i token sono esportati come custom properties organizzate per famiglie. Ecco gli import più comuni (da aggiungere al tuo entry CSS):

```30:49:README.md
@import '@primer/primitives/dist/css/base/size/size.css';
@import '@primer/primitives/dist/css/base/typography/typography.css';
@import '@primer/primitives/dist/css/functional/size/border.css';
@import '@primer/primitives/dist/css/functional/size/breakpoints.css';
@import '@primer/primitives/dist/css/functional/size/size.css';
@import '@primer/primitives/dist/css/functional/size/viewport.css';
@import '@primer/primitives/dist/css/functional/typography/typography.css';
@import '@primer/primitives/dist/css/functional/themes/light.css';
@import '@primer/primitives/dist/css/functional/themes/dark.css';
... (tutte le varianti high-contrast e colorblind)
```

Una volta importate, le variabili seguono la convenzione `--<token-path>` (es. `--bgColor-default`).

### JavaScript/TypeScript
Se preferisci lavorare con oggetti JS o TypeScript:

```ts
import tokens from '@primer/primitives/dist/json/tokens.json';

const borderRadius = tokens.functional.size.border.controlBorderRadius.value;
```

Puoi usare questi oggetti per generare sistemi di theming personalizzati (Emotion, Styled Components, Tailwind config, ecc.).

### JSON e automazioni
I file JSON/JSON5 esportati in `dist/json` sono utili per pipelines (es. generazione di documentazione, validazioni custom, sincronizzazione con tool interni). Il formato è piatto e segue la nomenclatura `category.name.state`.

### Storybook e Figma
- Storybook (`https://primer.style/primitives/storybook/`) offre anteprime delle palette, tipografia e motion.
- Il comando `npm run build:figma` produce l'estrazione dedicata per Figma, usando i metadati definiti in `$extensions`.

## Esempi pratici
### 1. Impostare tema light/dark con CSS
```css
/* theme.css */
@import '@primer/primitives/dist/css/functional/themes/light.css';
@import '@primer/primitives/dist/css/functional/themes/dark.css';

:root {
  color: var(--fgColor-default);
  background-color: var(--bgColor-default);
}

[data-color-mode='dark'] {
  color: var(--fgColor-default);
  background-color: var(--bgColor-default);
}
```
Associa `data-color-mode="dark"` al `body` o al container per passare da un tema all'altro.

### 2. Componenti React con token tipografici
```tsx
import '@primer/primitives/dist/css/base/typography/typography.css';
import '@primer/primitives/dist/css/functional/typography/typography.css';

export function HeroTitle({children}) {
  return (
    <h1 style={{
      fontFamily: 'var(--font-stack-system)',
      fontSize: 'var(--text-display-size)',
      lineHeight: 'var(--text-display-lineHeight)'
    }}>
      {children}
    </h1>
  );
}
```
In questo modo il componente eredita automaticamente qualsiasi aggiornamento sulle scale tipografiche.

### 3. Override mirati via token custom
```ts
import tokens from '@primer/primitives/dist/json/tokens.json';

export const marketingTheme = {
  ...tokens.functional.color,
  fgColor: {
    ...tokens.functional.color.fgColor,
    accent: {
      ...tokens.functional.color.fgColor.accent,
      default: '#3b82f6'
    }
  }
};
```
Combina i token forniti con valori personalizzati per esperimenti o superfici brandizzate, mantenendo comunque la struttura semantica.

### 4. Generare CSS custom con Style Dictionary
```sh
npx style-dictionary build --config sd.config.cjs
```
All'interno del config puoi puntare ai file JSON5 di `src/tokens` per produrre formati specifici (es. Android XML, iOS Swift). Riutilizza i transformer già presenti (`src/transformers`) per garantire compatibilità.

## Best practice
- **Importa solo ciò che usi**: suddividere gli import CSS riduce il peso e migliora i tempi di build.
- **Evita hardcode di colori**: fai riferimento ai token semantici (`fgColor.accent`, `bgColor.default`) per ottenere il supporto automatico ai temi accessibili.
- **Usa gli override** per gestire le varianti high-contrast o color-blind senza duplicare file.
- **Valida sempre** con `npm run lint:tokens` prima di aprire una pull request.
- **Documenta le aggiunte** aggiornando i metadati `$extensions` (scopes, utility classes, ecc.) per mantenere allineati Figma, CSS e Storybook.

Con questa guida puoi orientarti rapidamente tra i livelli dei token, capire il flusso di build e integrare le DS primitives nei tuoi progetti mantenendo coerenza con il design system di GitHub.
