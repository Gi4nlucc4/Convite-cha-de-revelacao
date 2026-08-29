# Convite digital — como montar, publicar e vender

Três arquivos, uma pasta, nenhum servidor. Você edita um bloco de configuração,
troca o vídeo e sobe.

```
convite/
├── index.html      ← o site inteiro (HTML + CSS + JS num arquivo só)
├── abertura.mp4    ← seu vídeo do Canva
└── poster.jpg      ← primeiro quadro do vídeo (evita tela preta no carregamento)
```

O `abertura.mp4` que veio junto é a **gravação de referência que você mandou**, só
para você ver o mecanismo funcionando. Troque pelo seu vídeo do Canva.

---

## 1. O bloco CONFIG

Abra o `index.html` num editor de texto e procure `const CONFIG = {`. É a única
parte que muda de festa para festa — está tudo lá em cima, comentado.

| Campo | O que é |
|---|---|
| `mae`, `pai`, `monograma` | Nomes e a letra que vai no lacre de cera |
| `quando` | Data e hora em ISO com fuso: `"2026-10-12T15:00:00-03:00"` |
| `local`, `endereco`, `mapa` | O `mapa` aceita qualquer link do Google Maps |
| `whatsapp` | Só números, com 55 + DDD. Ex.: `5551997044520` |
| `pix` | Deixe `""` para o bloco de Pix sumir da página |
| `video`, `musica` | Caminhos dos arquivos. `musica: ""` desliga a trilha |
| `estouro`, `faiscasDe`, `faiscasAte` | Sincronia dos efeitos (item 3) |

A data alimenta sozinha a contagem regressiva, o texto "Domingo, 12 de outubro
de 2026", o arquivo `.ics` da agenda e a mensagem do WhatsApp. Você digita uma
vez.

---

## 2. Exportando do Canva

No Canva: **Compartilhar → Baixar → MP4 Vídeo → Qualidade 1080p**.

Cuide de três coisas:

- **Formato vertical**, 1080×1920 ou 720×1280. O site usa `object-fit: cover`,
  então vídeo horizontal fica cortado nas laterais no celular.
- **Duração entre 8 e 15 segundos.** Trinta segundos é longo demais para quem
  abriu o convite no meio do trabalho. A referência tem 31s porque é uma
  gravação de tela, não a animação em si.
- **Termine no claro**, com a arte do convite já visível. O último quadro do
  vídeo continua na tela como fundo desfocado do convite — se o vídeo acabar em
  preto, a transição fica dura.

---

## 3. Sincronizando os efeitos com o seu vídeo

O site coloca por cima do vídeo: partículas douradas em canvas, um flash de luz
e um tremor curto da tela. Isso precisa cair no mesmo instante em que a fumaça
estoura no **seu** vídeo, senão parece desencontrado.

Assista o mp4 e anote os segundos. Depois ajuste:

```js
estouro:      2.4,   // segundo em que a fumaça sai — flash + tremor + vibração
faiscasDe:    2.2,   // partículas começam um pouco antes do estouro
faiscasAte:   9.0,   // e vão sumindo quando a fumaça dispersa
entregaAntes: 1.0,   // começa a virar o convite 1s antes do vídeo acabar
```

`entregaAntes` é o detalhe que separa amador de profissional: a transição para o
convite começa **antes** do vídeo terminar, então nunca aparece aquele frame
preto ou o controle nativo do player.

---

## 4. Comprimindo o vídeo

Vídeo pesado é convite que ninguém abre. A meta é **menos de 3 MB**. O Canva
costuma entregar 10–20 MB.

```bash
ffmpeg -i canva.mp4 \
  -vf "scale=720:-2" \
  -c:v libx264 -profile:v main -crf 26 -preset slow -pix_fmt yuv420p \
  -c:a aac -b:a 96k \
  -movflags +faststart \
  abertura.mp4
```

O que cada coisa resolve:

- `scale=720:-2` — 720px de largura é suficiente em tela de celular e corta o
  peso pela metade.
- `-pix_fmt yuv420p` — **obrigatório**. Sem isso o Safari do iPhone mostra tela
  preta e você só descobre no dia da festa.
- `-movflags +faststart` — joga o índice do arquivo para o começo, então o vídeo
  começa a tocar antes de baixar inteiro.
- `-crf 26` — qualidade. Aumente para 28 se ainda estiver pesado, baixe para 23
  se ficar borrado.

Se o vídeo não tiver áudio, não tem problema: o site trata os dois casos.

Depois gere o poster do primeiro quadro:

```bash
ffmpeg -i abertura.mp4 -frames:v 1 -q:v 3 poster.jpg
```

---

## 5. A prévia do WhatsApp

Convite se manda no WhatsApp. Se a prévia do link vier sem imagem, o convite
perde metade do impacto antes de ser aberto.

No topo do `index.html`, nas tags `og:`, troque `https://SEUDOMINIO.com.br/...`
pela URL real. A `og:image` tem que ser:

- link **absoluto** (`https://`), nunca `./capa.jpg`
- **1200×630 px**, JPG ou PNG
- menor que 300 KB

Faça essa capa no Canva também: o envelope lacrado, os nomes e "Toque para
abrir". Salve como `capa.jpg` na mesma pasta.

Depois de publicar, force o WhatsApp a reler a prévia mandando o link para você
mesmo. Se aparecer a capa antiga, acrescente `?v=2` no fim da URL.

---

## 6. Publicando

**Netlify Drop** é o caminho mais rápido: entre em `app.netlify.com/drop`,
arraste a pasta inteira, pronto. Sai um link tipo
`convite-marina.netlify.app`. Grátis, HTTPS incluso.

Para ficar profissional de verdade, registre um domínio curto
(`.site` ou `.com.br` saem barato) e aponte para lá. Aí cada cliente vira
uma pasta:

```
seudominio.com.br/marina-e-rafael
seudominio.com.br/cha-do-saulo
```

Copia a pasta, troca o CONFIG e o mp4, sobe. Uns dez minutos por convite depois
que o vídeo está pronto.

---

## 7. Checklist antes de entregar

- [ ] Abriu no **iPhone** (Safari) e no **Android** (Chrome) — não só no
      computador
- [ ] O vídeo tocou **com som** no primeiro toque, sem precisar de segundo clique
- [ ] O botão "Ativar som" apareceu só quando o navegador bloqueou o áudio
- [ ] Não piscou preto na virada do vídeo para o convite
- [ ] A contagem regressiva bate com a data certa
- [ ] O botão do WhatsApp abre a conversa **certa**, com a mensagem preenchida
- [ ] "Adicionar à agenda" baixou o `.ics` e o evento entrou no calendário
- [ ] Link do mapa cai no endereço certo
- [ ] Mandou o link no WhatsApp e a prévia veio com imagem
- [ ] Testou no 4G, não só no wi-fi

---

## 8. Detalhes que já estão resolvidos

Coisas que costumam quebrar em convite digital e que o arquivo já trata:

- **Áudio no iPhone.** O Safari só libera som dentro do mesmo toque do dedo. O
  site destrava o vídeo no clique, pausa, e só depois roda de verdade. Se ainda
  assim o navegador bloquear, aparece o botão "Ativar som" em vez de rodar mudo
  em silêncio.
- **Barra de progresso real.** O vídeo é baixado por streaming antes de tocar,
  então não engasga no meio. O botão só libera quando dá para assistir inteiro.
- **Plano B.** Se o mp4 não carregar (rede ruim, arquivo faltando, aberto direto
  do `file://`), entra uma fumaça desenhada em canvas e o convite abre do mesmo
  jeito. O convidado nunca vê erro.
- **Pular abertura.** Aparece depois de 4 segundos. Quem já viu não é obrigado a
  ver de novo.
- **Acessibilidade.** Quem liga "reduzir movimento" no celular recebe a versão
  sem tremor e sem partículas. Foco de teclado visível.
- **Bateria.** Sai da aba, o vídeo e a música pausam.
