## benchmark

> Esta landing page usa uma identidade visual escura, tecnica e premium. O site trabalha com:

# Design System Burd

## 1. Visao geral do design

Esta landing page usa uma identidade visual escura, tecnica e premium. O site trabalha com:

- fundo preto/carvao quase constante;
- linhas finas cinza-escuro estruturando a pagina como um grid editorial;
- blocos com imagens SVG grandes e composicoes densas;
- textos principais em sans serif estilo SF Pro;
- textos de apoio, labels e UI tecnica em mono (`JetBrains Mono`);
- contraste alto, mas sem branco puro dominante o tempo inteiro;
- azul e verde usados como acentos funcionais, nao como cor estrutural primaria.

A sensacao geral e de produto de infraestrutura/IA: frio, controlado, preciso, modular e orientado a sistema.

## 2. Principios visuais obrigatorios

- Manter dark mode como base de toda a interface.
- Usar bordas finas em `#262626` ou `#2A2A2A` para estrutura e divisorias.
- Manter blocos retangulares, grids e seccoes com composicao bem alinhada.
- Evitar cores fora da paleta extraida do projeto.
- Usar arredondamento minimo e seletivo. O sistema nao e baseado em cards fofos ou suaves.
- Usar mono em labels, microcopy, navegação tecnica, preços, FAQ auxiliar e indicadores.
- Evitar sombras fortes fora dos casos ja existentes de mock/cards sobre imagem.
- Evitar gradientes novos sem referencia direta nas artes atuais.
- Preservar a sensacao de layout arquitetado por linhas horizontais e verticais.
- Antes de criar uma nova interface, comparar o bloco novo com a landing atual em desktop e mobile.

## 3. Paleta de cores real do projeto

Valores extraidos diretamente de `app/page.tsx`, `app/globals.css` e classes utilitarias atuais.

| Token | Valor | Uso atual | Quando usar |
|---|---|---|---|
| `background.page` | `#0A0A0A` | fundo principal de `body`, `main`, cards escuros | fundo padrao de paginas e seccoes principais |
| `background.section.alt` | `#080808` | separadores vazios, bloco de texto central | secoes de pausa/transicao |
| `background.panel` | `#111111` | cards e blocos de preview | areas internas sobre o fundo principal |
| `background.panel.alt` | `#141414` | card central da secao azul | variacao sutil de card tecnico |
| `background.panel.deep` | `#090909` | `MockWindow` | UI embutida / painéis internos |
| `background.badge` | `#1A1A1A` | `SectionEyebrow`, estado ativo do nav glass | labels, pequenas barras, superficies compactas |
| `background.muted` | `#202020` | hover de botao | hover discreto |
| `border.default` | `#262626` | bordas principais da landing | divisorias gerais |
| `border.panel` | `#2A2A2A` | bordas do `MockWindow` | UI embutida, painéis detalhados |
| `border.muted` | `#2F2F2F` | barra vertical do eyebrow | acento estrutural secundario |
| `border.soft` | `#3A3A3A` | circulos/controles secundarios | elementos menos estruturais |
| `text.primary` | `#F5F5F5` | titulos, labels principais, nav | texto principal |
| `text.secondary` | `#9CA3AF` | paragrafos, labels tecnicas | texto auxiliar |
| `text.tertiary` | `#626262` | assinatura em depoimento | texto de hierarquia baixa |
| `text.inverse` | `#111111` | seta sobre faixa clara | texto sobre superficie clara |
| `accent.green` | `#3F8047` | status, ganhos positivos | status positivo, feedback funcional |
| `accent.blue` | `#1f7ea6` | elemento radial no card de escalabilidade | acentos graficos pontuais |
| `accent.brand.blue` | `#0091E2` | label do ecossistema em componente antigo | somente quando a composicao pedir um highlight institucional |
| `surface.light` | `#D9D9D9` | faixa da URL no mock | superficies claras de contraste alto |

Observacao: ha uso de opacidades recorrentes, como `opacity-70`, `opacity-90`, `opacity-40` e `opacity-[0.13]`, especialmente em imagens e overlays.

## 4. Tipografia

### Fontes

- Sans principal: `SF Pro Display`, `SF Pro Text`, `-apple-system`, `BlinkMacSystemFont`, `Inter`, `sans-serif`
- Mono: `JetBrains Mono`, `monospace`

### Regras

- Titulos principais usam sans.
- Textos auxiliares e informacoes tecnicas usam mono.
- Nao introduzir uma terceira familia tipografica.

### Escala real encontrada

| Uso | Classes / tamanho | Observacao |
|---|---|---|
| Hero H1 | `text-[32px] sm:text-[36px] xl:text-[40px]` | principal chamada institucional |
| H2 de secao | `text-[32px] xl:text-[40px]` | secoes principais |
| H3 de card | `text-[28px] xl:text-[32px]` | cards do ecossistema |
| Titulo FAQ item | `text-[18px] xl:text-[20px]` | perguntas |
| Label eyebrow | `text-[16px] uppercase` | mono, compacto |
| Nav desktop | `text-[16px]` | mono |
| Paragrafo padrao | `text-[14px] xl:text-[16px] leading-[1.6]` | copy de secao |
| Paragrafo tecnico pequeno | `text-[12px] xl:text-[11.8px] leading-[1.65]` | feature cards |
| Microcopy tecnica | `text-[10px]`, `text-[11px]`, `text-[13px]` | mocks, faq auxiliar, labels |
| Quote destaque | `text-[24px] xl:text-[32px] leading-[1.35]` | bloco editorial |

### Pesos e tracking

- O projeto usa principalmente peso normal/regular e bold visual via tamanho/contraste.
- Tracking custom aparece pontualmente, por exemplo `tracking-[-0.02em]` em bloco central e `tracking-[0.05em]` no mock.
- Nao exagerar em letter-spacing; usar apenas quando a referencia ja pedir isso.

## 5. Layout e grid

### Estrutura geral

- A pagina principal usa `GridShell`.
- Largura base: `w-[1540px] max-w-[1540px]` com `[zoom:0.8]` no wrapper interno.
- Ha uma borda vertical estrutural no lado direito do shell: `border-r border-[#262626]`.
- Ha uma linha vertical decorativa extra via `absolute ... w-px bg-zinc-800 xl:block`.

### Containers e paddings horizontais recorrentes

- mobile base: `px-8`
- small: `sm:px-10`
- desktop custom por secao: `md:px-[30px]`, `md:px-[33px]`, `md:px-[38px]`, `md:px-[43px]`, `md:px-[46px]`, `md:px-[62px]`, `md:px-[66px]`

### Padrões de grid

- Secoes 2 colunas: `xl:grid-cols-2`
- Secoes assimetricas: `xl:grid-cols-[701px_839px]`, `xl:grid-cols-[649px_891px]`, `xl:grid-cols-[651px_889px]`, `xl:grid-cols-[544px_996px]`
- Grades de features: `md:grid-cols-2 xl:grid-cols-4`

### Como manter alinhamento com a landing

- Sempre alinhar novas secoes aos mesmos paddings horizontais usados pelas secoes vizinhas.
- Se uma secao for split layout, considerar borda vertical entre colunas em desktop.
- Em mobile, empilhar primeiro e remover borda vertical, substituindo por `border-b` se necessario.

## 6. Espacamento

### Escala observada

- `mt-2`, `mt-3`, `mt-4`, `mt-5`, `mt-6`, `mt-8`, `mt-10`, `mt-12`
- `py-7`, `py-8`, `py-10`, `py-12`, `py-14`, `py-16`
- desktop grandes: `xl:py-[27px]`, `xl:py-[42px]`, `xl:py-[52px]`, `xl:py-[58px]`, `xl:py-[92px]`, `xl:py-[114px]`, `xl:py-[121px]`, `xl:py-[163px]`

### Separadores vazios

Padrao atual:

```tsx
<section
  aria-hidden="true"
  className="h-[117px] border-b border-[#262626] bg-[#080808]"
/>
```

Usar apenas para respiro entre secoes com conteudo forte. Nao colocar grid, linha vertical, `border-y` ou conteudo dentro.

### Min-height recorrente

- Hero: `h-[812px]`
- Cards principais: `xl:min-h-[472px]`
- Bloco workflow: `xl:min-h-[660px]`
- Footer: `min-h-[426px]`

## 7. Componentes base

### Button

Caracteristicas:

- retangular, baixo, tecnico;
- borda fina `#262626`;
- fundo `#1A1A1A`;
- texto claro;
- hover sutil.

Exemplo:

```tsx
function PrimaryButton({ label }: { label: string }) {
  return (
    <button className="inline-flex h-9 items-center border border-[#262626] bg-[#1A1A1A] px-3 text-[16px] text-[#F5F5F5] transition hover:border-[#333] hover:bg-[#202020]">
      <span>{label}</span>
      <span className="ml-3 inline-flex h-[29px] w-[29px] items-center justify-center">
        <Image src="/arrow.svg" alt="" width={29} height={29} />
      </span>
    </button>
  );
}
```

Nao fazer:

- botao muito alto;
- rounded grande;
- sombra exagerada;
- gradiente novo.

### Card

Tipos existentes:

- card de conteudo com imagem de fundo;
- card tecnico embutido (`MockWindow`);
- card de dados (`EarningsCard`).

Base recomendada:

```tsx
<div className="rounded-[4px] border border-[#262626] bg-[#111111]">
```

ou para UI interna:

```tsx
<div className="rounded-[8px] border border-[#2A2A2A] bg-[#090909]">
```

### Section

Base tipica:

```tsx
<section className="border-b border-[#262626]">
  <div className="px-8 py-16 sm:px-10 md:px-[38px] xl:py-[58px]">
    ...
  </div>
</section>
```

### Container

Para pagina inteira, reutilizar `GridShell`.
Para secoes internas, usar os paddings da secao vizinha em vez de inventar container novo.

### Badge / Eyebrow

Exemplo real:

```tsx
<div className="inline-flex h-8 w-fit max-w-max shrink-0 items-center self-start bg-[#1A1A1A] text-[16px] uppercase tracking-normal text-[#F5F5F5] pr-5">
  <span className="mr-4 h-8 w-1.5 shrink-0 bg-[#2F2F2F]" />
  <span className="whitespace-nowrap font-mono">PREÇOS</span>
</div>
```

### Divider

- horizontal principal: `border-b border-[#262626]`
- vertical principal: `border-r border-[#262626]` ou `border-l border-[#262626]`
- separador vazio: bloco de `117px`

### Header / Hero

Padrões:

- coluna esquerda com conteudo;
- coluna direita com imagem full;
- menu glass apenas em desktop;
- logo pequena e alinhada ao mesmo eixo do texto.

### Footer

Padrões:

- 2 colunas assimetricas em desktop;
- borda vertical central;
- linha abaixo da logo na coluna esquerda;
- menu no topo da direita;
- links legais e sociais no rodape da direita.

### FAQ Item

Exemplo real:

```tsx
<button
  type="button"
  aria-expanded={isOpen}
  aria-controls={`faq-answer-${index}`}
  className="flex w-full items-start justify-between gap-6 px-8 py-7 text-left xl:px-[32px] xl:py-[28px]"
>
```

### Pricing Card / Pricing Split

- texto fixo a esquerda;
- imagem dominante a direita;
- altura controlada, sem ocupar viewport inteira.

### Feature Card

- sem exagero visual;
- icone pequeno `h-4 w-4`;
- titulo curto;
- copy em mono;
- `+` decorativo nas intersecoes inferiores.

### Background Image Section

Padrão:

```tsx
<div className="relative h-[269px] w-full overflow-hidden">
  <Image fill className="object-cover object-center opacity-70" ... />
</div>
```

## 8. Padrao para novas secoes

### Secao com titulo e texto

```tsx
<section className="border-b border-[#262626] px-8 py-16 sm:px-10 md:px-[38px] xl:py-[92px]">
  <SectionEyebrow label="NOVA SECAO" />
  <h2 className="mt-[43px] max-w-[420px] text-[32px] leading-[1.2] xl:text-[40px]">
    Titulo da secao
  </h2>
  <p className="mt-6 max-w-[487px] font-mono text-[14px] leading-[1.6] text-[#9CA3AF] xl:text-[16px]">
    Texto explicativo.
  </p>
</section>
```

### Secao com cards

```tsx
<section className="border-b border-[#262626]">
  <div className="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-4">
    {items.map((item, index) => (
      <div
        key={item.title}
        className={`relative px-8 py-10 xl:px-[38px] xl:py-[52px] ${index < items.length - 1 ? "xl:border-r xl:border-[#262626]" : ""}`}
      >
        <div className="flex items-center gap-2.5">
          <item.icon className="h-4 w-4 text-[#F5F5F5]" strokeWidth={1.75} />
          <h3 className="text-[16px] leading-[1.25] text-[#F5F5F5]">{item.title}</h3>
        </div>
        <p className="mt-4 font-mono text-[12px] leading-[1.65] text-[#9CA3AF]">
          {item.text}
        </p>
      </div>
    ))}
  </div>
</section>
```

### Secao com imagem de fundo

```tsx
<section className="border-b border-[#262626] bg-[#111111] px-8 py-7 sm:px-10 md:px-[62px] xl:min-h-[472px] xl:py-[27px]">
  <div className="relative h-[269px] w-full overflow-hidden shadow-[0_4px_4px_rgba(0,0,0,0.25)]">
    <Image src="/BG Image 1.svg" alt="" fill className="object-cover object-center opacity-70" />
  </div>
</section>
```

### Secao em duas colunas

```tsx
<section className="border-b border-[#262626]">
  <div className="grid grid-cols-1 xl:grid-cols-[701px_839px]">
    <div className="border-b border-[#262626] xl:border-b-0 xl:border-r xl:border-[#262626]">...</div>
    <div>...</div>
  </div>
</section>
```

### Secao CTA

```tsx
<section className="relative border-b border-[#262626] px-8 py-16 text-center sm:px-10 md:px-14 xl:px-[92px] xl:py-[163px]">
  <Image src="/Ready Image.svg" alt="" fill className="object-cover opacity-40" />
  <div className="relative mx-auto max-w-[489px]">
    <h2 className="text-[32px] leading-[1.2] xl:text-[40px]">Titulo CTA</h2>
    <p className="mx-auto mt-6 max-w-[403px] font-mono text-[16px] leading-[1.3] text-[#9CA3AF] xl:text-[20px]">
      Texto CTA.
    </p>
  </div>
</section>
```

### Secao institucional

```tsx
<section className="relative border-b border-[#262626] px-8 py-16 text-center sm:px-10 md:px-14 xl:px-[131px] xl:py-[121px]">
  <Image src="/BG about us.svg" alt="" fill className="object-cover object-top opacity-[0.13]" />
  <div className="relative">
    <SectionEyebrow label="Institucional" />
    <blockquote className="mx-auto mt-[123px] max-w-[960px] text-[24px] leading-[1.35] xl:text-[32px]">
      Bloco editorial.
    </blockquote>
  </div>
</section>
```

### Secao de formulario

Inferido a partir do padrao atual:

```tsx
<section className="border-b border-[#262626] px-8 py-16 sm:px-10 md:px-[38px] xl:py-[58px]">
  <SectionEyebrow label="FORMULARIO" />
  <div className="mt-10 grid gap-4 xl:max-w-[520px]">
    <input className="h-12 border border-[#262626] bg-[#111111] px-4 font-mono text-[14px] text-[#F5F5F5]" />
    <textarea className="min-h-[160px] border border-[#262626] bg-[#111111] px-4 py-3 font-mono text-[14px] text-[#F5F5F5]" />
    <PrimaryButton label="Enviar" />
  </div>
</section>
```

## 9. Uso de assets

Assets atuais em `public/`:

- `Hero Image.svg`
- `BG Image 1.svg`
- `BG Image 2.svg`
- `BG Image 3.svg`
- `BG about us.svg`
- `Ready Image.svg`
- `Works Image 1.svg`
- `Coin Image.svg`
- `Llama Image.svg`
- `Docker.svg`
- `upper image.svg`
- `Cluster image.svg`
- `burd logo.svg`
- `arrow.svg`

Regras:

- Usar `next/image` para todas as imagens rasterizadas e SVGs da UI quando ja estiver nesse padrao.
- Em blocos de fundo, preferir `fill` + `object-cover`.
- Manter opacidade e crop coerentes com a landing.
- Evitar imagens pequenas demais dentro de cards grandes.
- Para assets novos, manter linguagem visual densa, escura, com textura tecnica ou abstrata coerente com IA/infra.
- Sempre validar o crop em desktop e mobile.

## 10. Responsividade

Padrão do projeto:

- mobile-first;
- `sm` para pequenos ajustes laterais;
- `md` para refinamento de paddings personalizados;
- `xl` para grids, larguras fixas, bordas verticais e composição final.

Regras:

- Em mobile, grids grandes viram coluna unica.
- Bordas verticais em desktop devem virar `border-b` ou desaparecer em mobile.
- Tamanhos de titulo principais geralmente sobem de `32px` para `40px` em `xl`.
- Paragrafos principais saem de `14px` para `16px` em `xl`.
- Nao deixar grandes vazios sem comparar com o ritmo da landing.

## 11. Regras de implementacao

- Nao criar estilos arbitrarios sem comparar com `app/page.tsx`.
- Nao usar cores fora dos tokens acima.
- Nao alterar espacamentos sem avaliar a secao anterior e a proxima.
- Reutilizar `SectionEyebrow`, `PrimaryButton` e padrões de split/grid sempre que possivel.
- Sempre validar mobile e desktop.
- Sempre checar se a nova tela parece parte da mesma landing.
- Manter classes Tailwind legiveis e organizadas por estrutura > espacamento > cor > efeito.
- Evitar helpers desnecessarios se a pagina atual ja resolve tudo inline.

## 12. Checklist antes de finalizar uma nova interface

- Conferiu se o fundo base e escuro?
- Conferiu se as bordas usam `#262626` ou `#2A2A2A`?
- Conferiu tipografia sans para titulos e mono para apoio/UI?
- Conferiu paddings horizontais consistentes com a landing?
- Conferiu responsividade em mobile e desktop?
- Conferiu se o bloco novo parece parte da landing atual?
- Conferiu header/footer alinhados com o sistema?
- Conferiu crops de imagem e opacidades?
- Conferiu divisorias horizontais/verticais?
- Rodou `npm run build`?

## 13. Exemplos prontos

### Nova pagina institucional

```tsx
import Image from "next/image";

export default function SobrePage() {
  return (
    <main className="overflow-x-hidden bg-[#0A0A0A] text-[#F5F5F5]">
      <section className="border-b border-[#262626] px-8 py-16 sm:px-10 md:px-[38px] xl:py-[92px]">
        <SectionEyebrow label="SOBRE" />
        <h1 className="mt-[43px] max-w-[520px] text-[32px] leading-[1.2] xl:text-[40px]">
          Infraestrutura local para IA com linguagem de produto global
        </h1>
        <p className="mt-6 max-w-[487px] font-mono text-[14px] leading-[1.6] text-[#9CA3AF] xl:text-[16px]">
          Texto institucional mantendo o mesmo tom tecnico e premium.
        </p>
      </section>
    </main>
  );
}
```

### Nova secao de cards

```tsx
const items = [
  { title: "Latencia local", text: "Resposta mais rapida para workloads na America Latina.", icon: Sparkles },
  { title: "Escolha tecnica", text: "Compare custo, benchmark e uptime antes de subir.", icon: SquaresUnite },
  { title: "Visibilidade", text: "Painel com leitura clara de uso e credito.", icon: Flower },
  { title: "Escala", text: "Expanda sem lock-in de cloud tradicional.", icon: TrendingUp },
];

<section className="border-b border-[#262626]">
  <div className="relative grid grid-cols-1 border-t border-[#262626] md:grid-cols-2 xl:grid-cols-4">
    {items.map((item, index) => (
      <div
        key={item.title}
        className={`relative min-h-[180px] px-8 py-10 sm:px-10 xl:min-h-[232px] xl:px-[38px] xl:py-[52px] ${index < items.length - 1 ? "xl:border-r xl:border-[#262626]" : ""}`}
      >
        <div className="flex items-center gap-2.5">
          <item.icon className="h-4 w-4 text-[#F5F5F5]" strokeWidth={1.75} />
          <h3 className="text-[16px] leading-[1.25] text-[#F5F5F5]">{item.title}</h3>
        </div>
        <p className="mt-4 font-mono text-[12px] leading-[1.65] text-[#9CA3AF]">{item.text}</p>
      </div>
    ))}
  </div>
</section>
```

### Nova secao CTA

```tsx
<section className="relative border-b border-[#262626] px-8 py-16 text-center sm:px-10 md:px-14 xl:px-[92px] xl:py-[163px]">
  <Image src="/Ready Image.svg" alt="" fill className="object-cover opacity-40" />
  <div className="relative mx-auto max-w-[489px]">
    <h2 className="text-[32px] leading-[1.2] xl:text-[40px]">Comece com um workload simples</h2>
    <p className="mx-auto mt-6 max-w-[403px] font-mono text-[16px] leading-[1.3] text-[#9CA3AF] xl:text-[20px]">
      Suba, compare e pague em reais sem depender de cloud estrangeira.
    </p>
    <div className="mt-6 flex justify-center">
      <PrimaryButton label="Explorar" />
    </div>
  </div>
</section>
```

### Novo bloco de FAQ

```tsx
const [openIndex, setOpenIndex] = useState(0);

<div className="border-2 border-[#262626]">
  {items.map((item, index) => {
    const isOpen = openIndex === index;

    return (
      <div key={item.question} className={index < items.length - 1 ? "border-b border-[#262626]" : ""}>
        <button
          type="button"
          onClick={() => setOpenIndex(index)}
          aria-expanded={isOpen}
          className="flex w-full items-start justify-between gap-6 px-8 py-7 text-left xl:px-[32px] xl:py-[28px]"
        >
          <div>
            <h3 className="text-[18px] leading-[1.2] xl:text-[20px]">{item.question}</h3>
            {isOpen ? (
              <p className="mt-4 max-w-[702px] font-mono text-[13px] leading-[1.55] text-[#9CA3AF] xl:text-[14px]">
                {item.answer}
              </p>
            ) : null}
          </div>
          <span className="shrink-0 text-[28px] leading-none text-white xl:text-[40px]">{isOpen ? "-" : "+"}</span>
        </button>
      </div>
    );
  })}
</div>
```

### Novo layout com imagem e texto

```tsx
<section className="border-b border-[#262626]">
  <div className="grid grid-cols-1 xl:grid-cols-[649px_891px]">
    <div className="px-8 py-16 sm:px-10 md:px-[38px] xl:flex xl:min-h-[420px] xl:flex-col xl:justify-center xl:py-0">
      <SectionEyebrow label="INFRA" />
      <h2 className="mt-[43px] max-w-[369px] text-[32px] leading-[1.2] xl:text-[40px]">
        Compare maquinas com criterio tecnico
      </h2>
      <p className="mt-[43px] max-w-[487px] font-mono text-[14px] leading-[1.6] text-[#9CA3AF]">
        Texto com o mesmo ritmo visual da secao de precos.
      </p>
    </div>

    <div className="relative min-h-[340px] overflow-hidden xl:min-h-[420px] xl:border-l xl:border-[#262626]">
      <Image src="/Coin Image.svg" alt="" fill className="scale-[1.18] object-cover object-[55%_28%] opacity-90" />
    </div>
  </div>
</section>
```

## 14. Como usar este SKILL.md

Fluxo recomendado para qualquer pessoa ou agente de IA:

1. Ler este `SKILL.md` inteiro.
2. Abrir `app/page.tsx` para comparar com os padrões reais.
3. Identificar qual bloco da landing mais se parece com a nova necessidade.
4. Reutilizar componentes, escalas e classes existentes antes de criar algo novo.
5. Construir a interface mantendo paleta, bordas, espaçamento e tipografia.
6. Comparar visualmente com a landing em desktop e mobile.
7. Rodar `npm run build` antes de finalizar.

Se houver ambiguidade, preferir a leitura mais conservadora da landing atual. Este design system existe para preservar a identidade visual do site, nao para reinventar a interface.

---
> Source: [Burd-repo/benchmark](https://github.com/Burd-repo/benchmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
