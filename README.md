# Quiz "Teste de Narrativa Visual" (Luísa Reis)

É um arquivo só: `index.html`. Fotos, logo, textos e regras estão todos dentro dele. Ele funciona em qualquer hospedagem e abre direto no navegador.

## Como funciona (versão de bolso)

1. **Tela de abertura:** a pessoa vê a pergunta "Sua imagem vende antes do seu discurso?" e começa.
2. **10 perguntas de diagnóstico:** são 2 para cada um dos 5 códigos da narrativa visual, que são os 5 erros que a Luísa vê nos perfis. Cada resposta vale 2, 1 ou 0 pontos, então a nota máxima é 20.
3. **4 perguntas sobre o momento da pessoa:** área, faturamento, marketing e objetivo. O faturamento separa quem é cliente ideal (R$ 30 mil ou mais por mês) de quem ainda não é.
4. **Contato:** nome, WhatsApp, @ do Instagram e e-mail (opcional).
5. **"Analisando…":** uma pausa curta, que faz o resultado parecer mais trabalhado.
6. **Resultado:**
   - nota de 0 a 20;
   - perfil da imagem: genérica, desalinhada ou quase premium;
   - as 5 barras, com os 2 pontos mais fracos destacados;
   - o primeiro ajuste para cada ponto fraco;
   - a tabela "Padrão × Narrativa visual";
   - o antes e depois do case;
   - o convite para a reunião;
   - quem é a Luísa;
   - perguntas frequentes.
7. **Botão final:**
   - **Qualificado:** abre o WhatsApp da Luísa (22 99731-5303) com a mensagem já escrita. A mensagem traz o nome, o @, a área, o faturamento, o objetivo, a nota, os pontos fracos e a origem do anúncio.
   - **Não qualificado:** o botão principal leva ao Instagram dela, com um link menor "Quero conversar com a Luísa mesmo assim".

Se a pessoa fechar no meio e voltar depois, o quiz continua de onde parou. O botão "voltar" do celular também funciona dentro do quiz.

## Onde mexer: o bloco CONFIG

Abra o `index.html` num editor de texto, procure por `var CONFIG` e altere só o que está entre aspas:

| Campo | O que é | Hoje |
|---|---|---|
| `waNumber` | WhatsApp que recebe os leads (55 + DDD + número, só dígitos) | `5522997315303` |
| `igUser` | @ do Instagram, sem o @ | `luisaareiss` |
| `pixelId` | ID do Pixel da Meta | vazio, falta criar |
| `leadWebhook` | Link da planilha que recebe os leads (passo a passo abaixo) | vazio |
| `qualifiedCodes` | Faixas de faturamento consideradas qualificadas | R$ 30–100 mil e R$ 100 mil+ |
| `spotsPerMonth` | Vagas por mês, que aparece no convite | `5` |
| `spotsTaken` | Vagas já ocupadas no mês. Se ficar `null`, o contador não aparece | `null` |
| `showCase` | Mostrar ou não o antes e depois | `true` |

Sobre o `spotsTaken`: só preencha se o número for real e se alguém for atualizar todo mês.

## 1. Pixel (quando você criar)

1. No Gerenciador de Eventos, crie o pixel e copie o ID (só números).
2. Cole o ID em `pixelId: '...'` e salve.
3. Pronto. O quiz já dispara estes eventos sozinho:

| Evento | Quando dispara | Tipo |
|---|---|---|
| `PageView` | Abriu a página | padrão |
| `QuizStart` | Clicou em "Fazer meu diagnóstico" | personalizado |
| `QuizStep` | A cada resposta | personalizado |
| `AxisComplete` | Concluiu cada código | personalizado |
| `QuizComplete` | Terminou as 14 perguntas | personalizado |
| `Lead` | Enviou o contato. Leva o status qualificado ou não qualificado e já manda telefone, nome e e-mail para a Meta (correspondência avançada) | padrão |
| `QualifiedLead` | Lead com faturamento de R$ 30 mil ou mais | personalizado |
| `ViewResult` | Viu o resultado | personalizado |
| `Contact` e `WhatsAppClick` | Clicou para falar no WhatsApp | padrão e personalizado |
| `InstagramClick` | Foi para o Instagram | personalizado |
| `QuizResumed` | Voltou depois e retomou | personalizado |

**Sugestão de otimização:**
- Comece a campanha otimizando para **Lead**.
- Quando juntar uns 50 `QualifiedLead` por semana, crie uma conversão personalizada com esse evento e teste otimizar para ela. Assim a Meta procura mais gente parecida com quem fatura R$ 30 mil ou mais.

## 2. Planilha de leads (importante)

Sem isso, **só chega lead para a Luísa quando a pessoa clica no WhatsApp**. Quem preenche o contato e fecha a página se perde. Com a planilha, todo mundo que preencher cai numa linha nova, com nota, área, faturamento e a origem do anúncio.

Passo a passo (uns 5 minutos):
1. Crie uma Planilha Google em branco.
2. Vá em **Extensões → Apps Script**, apague o que estiver lá e cole o código abaixo:

```javascript
function doPost(e) {
  var sh = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
  var d = JSON.parse(e.postData.contents);
  var cols = ['data','nome','whatsapp','instagram','email','area','faturamento','marketing',
              'objetivo','pontuacao','perfil','pontos_fracos','codigos','qualificado','origem'];
  if (sh.getLastRow() === 0) sh.appendRow(cols.concat(['utm_source','utm_campaign','utm_content']));
  var u = d.utm || {};
  sh.appendRow(cols.map(function (c) { return d[c] === undefined ? '' : d[c]; })
    .concat([u.source || '', u.campaign || '', u.content || '']));
  return ContentService.createTextOutput('ok');
}
```

3. Clique em **Implantar → Nova implantação → tipo "App da Web"** e preencha:
   - Executar como: **Eu**
   - Quem pode acessar: **Qualquer pessoa**
4. Autorize, copie o link que termina em `/exec` e cole em `leadWebhook: '...'`.
5. Faça o quiz uma vez você mesmo e confira se a linha apareceu na planilha.

Dica: na planilha, ative uma notificação de alterações (Ferramentas → Regras de notificação) para receber um e-mail a cada lead novo.

## 3. Origem do anúncio (UTM)

Nos parâmetros de URL do anúncio, cole:

```
utm_source=meta&utm_campaign={{campaign.name}}&utm_content={{ad.name}}
```

O nome do anúncio vai aparecer na mensagem do WhatsApp ("Ref: …") e na planilha. Assim, a Luísa sabe qual criativo trouxe cada reunião. Para testar à mão, dá para usar algo como `?ref=roteiroB`.

## 4. Onde hospedar

Qualquer lugar que aceite um arquivo HTML serve: Netlify Drop (é só arrastar o arquivo), Vercel, Hostinger ou um subdomínio dela, como `teste.luisareis.com.br`.

Se usar domínio próprio, verifique o domínio no Gerenciador de Negócios da Meta.

## 5. O que a Luísa precisa confirmar antes de subir

1. **Antes e depois do Arthur Lessa:** o nome e o @ dele aparecem nas imagens. Ela precisa ter a autorização dele. Se não tiver, mude `showCase` para `false`.
2. **Textos que escrevi na voz dela:** o "primeiro ajuste" de cada código, o que acontece na reunião e as perguntas frequentes. Vale ela ler e trocar o que não soar como ela.
3. **As 10 perguntas:** ver se ela se reconhece nas opções. Se quiser trocar alguma, me manda que eu ajusto.
4. **A "condição exclusiva para quem decide na reunião":** o quiz cita isso sem dizer o valor. Confirmar se ela quer manter.
5. **Política de privacidade:** o formulário já tem o aviso da LGPD. Se ela tiver uma página de política, dá para colocar o link.
