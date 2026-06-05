# Perf Fiscal — Lint de performance para código em escala

[![npm version](https://img.shields.io/npm/v/eslint-plugin-perf-fiscal.svg?color=informational)](https://www.npmjs.com/package/eslint-plugin-perf-fiscal)
[![build](https://img.shields.io/badge/build-tsc%20--p%20tsconfig.build-blue)](#fluxo-de-desenvolvimento)
[![license](https://img.shields.io/github/license/ruidosujeira/perf-linter.svg)](LICENSE)
![Cross-File Powered](https://img.shields.io/badge/Intelig%C3%AAncia-Cross--File-blueviolet?style=flat-square)
![Rust Core](https://img.shields.io/badge/Core-Rust-orange?style=flat-square)

Slogan: Entregue rápido. Continue rápido.

Perf Fiscal é um plugin ESLint profissional que leva a disciplina de um engenheiro de performance para cada revisão de código. Ele entende seu código inteiro (análise cross-file), fala React fluentemente e aproveita um core em Rust para velocidade e precisão.

## Sumário

- [Por que Perf Fiscal](#por-que-perf-fiscal)
- [Novidades](#novidades)
- [Primeiros Passos](#primeiros-passos)
- [Core em Rust](#core-em-rust)
- [Inteligência Cross-File](#intelig%C3%AAncia-cross-file)
- [Destaques de Configuração](#destaques-de-configura%C3%A7%C3%A3o)
- [Catálogo de Regras](#cat%C3%A1logo-de-regras)
- [Exemplos Guiados](#exemplos-guiados)
- [Compatibilidade](#compatibilidade)
- [Fluxo de Desenvolvimento](#fluxo-de-desenvolvimento)
- [Como Contribuir](#como-contribuir)
- [Licença](#licen%C3%A7a)
- [Fique por Dentro](#fique-por-dentro)

## Por que Perf Fiscal

- Visão de base inteira: entende componentes, props, fluxos assíncronos e imports além dos limites do módulo.
- Foco em React: protege fronteiras de memo, arrays de dependência e estabilidade de Context com sugestões acionáveis.
- Regras orientadas a performance: captura loops pesados, crescimento quadrático, operações de string custosas e armadilhas de bundle.
- Importações conscientes: detecta entrypoints pesados e sugere subpaths ou alternativas mais leves com segurança.
- Aceleração em Rust: core opcional em Rust para parsing, indexação e checagens de segurança, com fallback seguro em JS.
- Baixa fricção: presets flat e clássicos do ESLint; nenhum setup obrigatório além do seu TS atual.

## Inteligência Cross-File

- 🔍 Analyzer de projeto inteiro: indexa exports, wrappers de memo e assinaturas de props esperadas (function | object | array | primitive) para cada componente React, reduzindo falsos positivos.
- 🙌 `no-unstable-inline-props` com contexto: relaxa avisos para componentes não memoizados e alinha diagnósticos ao tipo declarado da prop.
- 🛟 `no-unhandled-promises` tipado: reconhece helpers que retornam Promise importados de outros módulos em vez de depender de nomes.
- 🧱 Infraestrutura extensível: regras consultam metadata compartilhada via `getCrossFileAnalyzer`, habilitando heurísticas que entendem o grafo do projeto inteiro.

O Perf Fiscal rastreia fronteiras de memo, tipos de prop e fluxos assíncronos entre arquivos — entregando diagnósticos mais inteligentes e de baixo ruído que escalam.

### Captura de Alerta Cross-File

```text
tests/fixtures/cross-file/consumer.tsx:21:7
  21:7  warning  perf-fiscal/no-unhandled-promises  Unhandled Promise: await this call or return/chain it to avoid swallowing rejections.
          • Origin: useDataSource (exported from tests/fixtures/cross-file/components.tsx)
```

Esse diagnóstico rastreia o helper assíncrono até o arquivo de origem, provando que o analyzer entende fronteiras de memoização e fluxos assíncronos além do módulo atual.

## Saída de Exemplo

Ao executar `perf-fiscal/no-unstable-inline-props`, você verá feedback contextual como:

```text
src/pages/Profile.tsx:12:13: [perf-fiscal/no-unstable-inline-props] Passing inline function to memoized child <Child onSelect={...}/> — wrap in useCallback for stable renders (expected prop kind: function)
```

E para detecção de fluxos assíncronos cross-file:

```text
src/utils/api.ts:8:5: [perf-fiscal/no-unhandled-promises] Unhandled Promise returned from helper `fetchUserData` (imported from utils/http.ts) — consider awaiting or handling rejections.
```

Esses exemplos mostram como os diagnósticos enriquecidos trazem a origem e o tipo esperado de prop, acelerando correções com confiança.

## Novidades

- ⚡ Grafo de Metadata Cross-File (Rust): indexe seu projeto em paralelo e exponha metadados de componentes/props/imports/exports via um snapshot JSON rápido.
- 🧩 Parser em Rust (SWC): novo CLI `parse` para JS/TS/JSX/TSX com ponte TypeScript e fallbacks elegantes.
- 🦀 Guarda de segurança em Rust: `check-redos` reforça a regra `no-redos-regex` com I/O em JSON e timeouts.
- 📦 Higiene de imports: `no-heavy-bundle-imports` ajuda a evitar entrypoints monolíticos; docs atualizados com racional e opções.
- 🛡️ Estabilidade em React: `no-inline-context-value` previne churn em Context antes que se espalhe pelo app.

Veja as notas completas em [docs/changelog/0.5.0.md](docs/changelog/0.5.0.md). Para manter o comportamento anterior, deixe `debugExplain` no padrão (`false`) ou desligue por regra:

```json
{
  "perf-fiscal/no-unhandled-promises": ["warn", { "debugExplain": false }]
}
```

Encontrou regressão ou alerta barulhento? Abra o [template de False Positive](https://github.com/ruidosujeira/perf-linter/issues/new?template=false-positive.md) para agilizar o triagem.

## Primeiros Passos

> 🧭 **Quer diagnósticos tipados?** Consulte o guia [Configuração do Analyzer Tipado](docs/typed-analyzer-setup.md). Resumo:
> (1) crie um `tsconfig` dedicado ao lint que inclua todos os arquivos relevantes, (2) aponte `parserOptions.project`/`tsconfigRootDir`
> para esse arquivo e (3) mantenha `@typescript-eslint/parser` alinhado à versão do ESLint. Se o ESLint acusar "Cannot read file
> 'tsconfig...json'" ou "parserServices to be generated", revise as orientações de `tsconfigRootDir` descritas no guia.

### Instalação

```bash
npm install --save-dev eslint eslint-plugin-perf-fiscal
# ou
yarn add --dev eslint eslint-plugin-perf-fiscal
# ou
pnpm add -D eslint eslint-plugin-perf-fiscal
```

## Core em Rust

O Perf Fiscal pode, opcionalmente, usar um core leve em Rust para velocidade e precisão. Quando indisponível, o plugin faz fallback de forma transparente para as implementações em JS.

Compile uma vez (local ou CI):

```bash
cd rust/perf-linter-core
cargo build --release
```

Opcionalmente, aponte o plugin para o binário (se não estiver no caminho padrão):

```bash
export PERF_LINTER_CORE="$(pwd)/target/release/perf-linter-core"
```

Comandos e pontes disponíveis:

- ReDoS checker: `perf-linter-core check-redos` (STDIN `{ "pattern": string }` → STDOUT `{ "safe": boolean, "rewrite"?: string }`)
- Parser (SWC): `echo "const x=1" | perf-linter-core parse --filename input.tsx`
- Indexador de projeto: `perf-linter-core index /caminho/do/projeto > metadata.json`

Pontes em TypeScript:

- Ponte do parser: `src/utils/rust-parser.ts` (`parseWithRust(source, filename)` com cache + timeout)
- Ponte do analyzer cross-file: `src/analyzer/cross-file.ts` (`getCrossFileAnalyzer(projectRoot)` com cache em arquivo + memória)

### Config Flat (ESLint ≥8.57)

```js
import perfFiscal from 'eslint-plugin-perf-fiscal';

const tsParser = await import('@typescript-eslint/parser');

export default [
  {
    files: ['**/*.ts', '**/*.tsx'],
    languageOptions: {
      parser: tsParser.default,
      parserOptions: {
        project: ['./tsconfig.json'],
        tsconfigRootDir: import.meta.dirname
      }
    }
  },
  perfFiscal.configs.recommended
];
```

Nota: o analyzer cross-file se beneficia de configurações com conhecimento do projeto (`parserOptions.project` + `tsconfigRootDir`) para consultar o checker do TypeScript e seguir símbolos entre arquivos.

## Guias de Migração

Pronto para adotar o Perf Fiscal em um projeto existente? Escolha o guia que combina com sua arquitetura:

- [Guia de Migração para Aplicações React](docs/migrations/react.md) – organize o rollout em apps React e React Native mantendo a estabilidade das memoizações.
- [Guia de Migração para Serviços Node.js](docs/migrations/node-services.md) – integre o plugin em backends, CLIs e workers que dependem de fluxos assíncronos confiáveis.
- [Guia de Migração para Monorepos Híbridos](docs/migrations/monorepo.md) – coordene a adoção entre workspaces que misturam frontends, serviços e pacotes compartilhados.

Cada guia oferece um passo a passo de adoção, trechos de configuração e notas de compatibilidade específicas para cada cenário.

### Config Clássico (`.eslintrc.*`)

```js
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: ['./tsconfig.json'],
    tsconfigRootDir: __dirname
  },
  extends: ['plugin:perf-fiscal/recommended']
};
```

### Habilitando Regras Específicas

```js
module.exports = {
  plugins: ['perf-fiscal'],
  rules: {
    'perf-fiscal/no-expensive-split-replace': 'warn',
    'perf-fiscal/prefer-array-some': 'error',
    'perf-fiscal/no-unstable-inline-props': ['warn', {
      ignoreProps: ['className'],
      checkSpreads: false
    }]
  }
};
```

## Catálogo de Regras

Cada regra possui documentação detalhada em `docs/rules/<nome-da-regra>.md`.

| Regra | Detecta | Ação Recomendada | Documentação |
| --- | --- | --- | --- |
| `perf-fiscal/detect-unnecessary-rerenders` | 🚦 Handlers inline passados para filhos memoizados | Extraia callbacks ou use `useCallback` | [docs/rules/detect-unnecessary-rerenders.md](docs/rules/detect-unnecessary-rerenders.md) |
| `perf-fiscal/no-expensive-computations-in-render` | 🧮 Trabalho síncrono pesado durante renderizações | Movê-lo para `useMemo` ou fora do componente | [docs/rules/no-expensive-computations-in-render.md](docs/rules/no-expensive-computations-in-render.md) |
| `perf-fiscal/no-expensive-split-replace` | 🔁 `split`/`replace` repetidos em loops quentes | Pré-computar e reutilizar resultados | [docs/rules/no-expensive-split-replace.md](docs/rules/no-expensive-split-replace.md) |
| `perf-fiscal/no-heavy-bundle-imports` | 📦 Imports default de pacotes pesados (`lodash`, `moment`, SDKs legados) | Migrar para subpaths ou alternativas leves | [docs/rules/no-heavy-bundle-imports.md](docs/rules/no-heavy-bundle-imports.md) |
| `perf-fiscal/no-inline-context-value` | 🫧 Objetos/arrays inline em `Context.Provider value` | Envolver em `useMemo` ou extrair fora do render | [docs/rules/no-inline-context-value.md](docs/rules/no-inline-context-value.md) |
| `perf-fiscal/no-quadratic-complexity` | 🧮 Loops aninhados de crescimento quadrático | Refatorar ou pré-indexar coleções | [docs/rules/no-quadratic-complexity.md](docs/rules/no-quadratic-complexity.md) |
| `perf-fiscal/no-unhandled-promises` | ⚠️ Promises ignoradas | `await` ou encadear `.catch`/`.then` | [docs/rules/no-unhandled-promises.md](docs/rules/no-unhandled-promises.md) |
| `perf-fiscal/no-unstable-inline-props` | ✋ Funções/objetos inline e spreads que mudam referências | Memorizar antes de passar como prop | [docs/rules/no-unstable-inline-props.md](docs/rules/no-unstable-inline-props.md) |
| `perf-fiscal/no-unstable-usememo-deps` | 🧩 Valores instáveis em arrays de dependência | Memorizar dependências ou movê-las para fora do render | [docs/rules/no-unstable-usememo-deps.md](docs/rules/no-unstable-usememo-deps.md) |
| `perf-fiscal/prefer-array-some` | ✅ `filter(...).length` usado para checar existência | Trocar por `Array.prototype.some` | [docs/rules/prefer-array-some.md](docs/rules/prefer-array-some.md) |
| `perf-fiscal/prefer-for-of` | 🔄 Uso de `map`/`forEach` apenas por efeitos colaterais | Migrar para `for...of` para clareza e performance | [docs/rules/prefer-for-of.md](docs/rules/prefer-for-of.md) |
| `perf-fiscal/prefer-object-hasown` | 🧾 Padrões legados com `hasOwnProperty.call` | Usar `Object.hasOwn` | [docs/rules/prefer-object-hasown.md](docs/rules/prefer-object-hasown.md) |
| `perf-fiscal/prefer-promise-all-settled` | 🤝 `Promise.all` esperando falhas parciais | Migrar para `Promise.allSettled` | [docs/rules/prefer-promise-all-settled.md](docs/rules/prefer-promise-all-settled.md) |
| `perf-fiscal/vue-no-expensive-computed` | 🧮 Getters `computed` do Vue caros ou complexos demais | Dividir o getter ou tirar trabalho pesado da reatividade | [docs/rules/vue-no-expensive-computed.md](docs/rules/vue-no-expensive-computed.md) |
| `perf-fiscal/vue-no-inefficient-watchers` | 👁️ Watchers Vue profundos, aninhados ou apenas derivativos | Observar fontes específicas ou migrar para `computed` | [docs/rules/vue-no-inefficient-watchers.md](docs/rules/vue-no-inefficient-watchers.md) |
| `perf-fiscal/vue-optimize-reactivity` | ⚡ Uso indevido de `reactive()`/`ref()` e reatividade criada em loops | Usar `ref` para primitivos e tirar reatividade de dentro de loops | [docs/rules/vue-optimize-reactivity.md](docs/rules/vue-optimize-reactivity.md) |

> As regras Vue fazem parte dos presets `vue` / `flat/vue`. Veja [Configuração](#destaques-de-configura%C3%A7%C3%A3o) para habilitá-las.

## Destaques de Configuração

- 🧰 **Presets flat vs. clássicos:** Use `perfFiscal.configs.recommended` em configs flat ou `plugin:perf-fiscal/recommended` em configs clássicas.
- 🛰️ **Habilite a inteligência cross-file:** Configure `@typescript-eslint/parser` com `parserOptions.project` e `tsconfigRootDir` para que o Perf Fiscal possa invocar o checker do TypeScript e seguir símbolos entre arquivos.
- 🧭 **Controle de severidade:** Ajuste severidades (`off`, `warn`, `error`) conforme sua política interna.
- ⚙️ **Opções de regra:** Algumas regras expõem configurações específicas. Consulte a documentação de cada regra para detalhes. Exemplo:

  ```js
  'perf-fiscal/no-unstable-inline-props': ['warn', {
    ignoreProps: ['className', 'data-testid'],
    checkFunctions: true,
    checkObjects: true,
    checkSpreads: true
  }],
  'perf-fiscal/no-heavy-bundle-imports': ['warn', {
    packages: [
      { name: 'lodash', suggestSubpath: true },
      { name: '@org/legacy-sdk', allowNamed: true }
    ]
  }]
  ```
- 🧮 **Presets de rigor de performance:** As regras mais ruidosas agora compartilham opções como `strictness` (`relaxed` \| `balanced` \| `strict`), `includeTestFiles`, `includeStoryFiles` e `debugExplain`. Use-as para controlar o nível de ruído, ignorar pastas de fixtures ou exibir pistas de confiança:

  ```js
  'perf-fiscal/no-expensive-computations-in-render': ['warn', {
    strictness: 'strict',
    includeTestFiles: false,
    debugExplain: true
  }],
  'perf-fiscal/no-expensive-split-replace': ['warn', { strictness: 'relaxed' }],
  'perf-fiscal/no-unhandled-promises': ['error', { strictness: 'balanced' }]
  ```

## Exemplos Guiados

### Estabilize Callbacks em React

```tsx
// Antes: callbacks recriados a cada render
const Parent = () => <Child onSelect={() => dispatch()} />;

// Depois: identidades estáveis
const Parent = () => {
  const onSelect = useCallback(() => dispatch(), []);
  return <Child onSelect={onSelect} />;
};
```

### Faça Cache de Operações de String Pesadas

```ts
// Antes: split custoso por item
for (const record of records) {
  const parts = record.path.split('/');
  visit(parts);
}

// Depois: compute uma vez
const parts = basePath.split('/');
for (const record of records) {
  visit(parts);
}
```

### Memorize Objetos Antes de Espalhar Props

```tsx
// Antes: spread gera referências instáveis
const Panel = ({ onSubmit }) => <Form {...{ onSubmit: () => onSubmit() }} />;

// Depois: payload memoizado
const Panel = ({ onSubmit }) => {
  const formProps = useMemo(() => ({ onSubmit: () => onSubmit() }), [onSubmit]);
  return <Form {...formProps} />;
};
```

### Memorize Valores de Context Providers

```tsx
// Antes: objeto inline invalida todos os consumidores a cada render
return (
  <UserContext.Provider value={{ name, role, refresh: () => refetch() }}>
    <Profile />
  </UserContext.Provider>
);

// Depois: memoize para manter o Context estável
const providerValue = useMemo(() => ({ name, role, refresh: () => refetch() }), [name, role, refetch]);
return (
  <UserContext.Provider value={providerValue}>
    <Profile />
  </UserContext.Provider>
);
```

### Evite Entrypoints Pesados

```ts
// Antes: importa todo o build do lodash
import { map } from 'lodash';

// Depois: traga apenas o necessário
import map from 'lodash/map';
```
