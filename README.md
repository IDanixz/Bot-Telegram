# Bot Shopee → Telegram

Busca produtos na API de Afiliados da Shopee e posta automaticamente no seu
canal do Telegram, no formato:

```
Nome do produto


💰 Preço

🔗 Link (afiliado)


Nome do canal
Link do canal
```

## 1. Pré-requisitos

- Python 3.9 ou superior instalado ([python.org](https://www.python.org/downloads/))
- Conta aprovada no **Shopee Affiliate Open Platform**, com `App ID` e `Secret`
- Um bot no Telegram (ver passo 3)

## 2. Instalação

1. Extraia o `.zip`/`.rar` em uma pasta no seu computador.
2. Abra o terminal (CMD/PowerShell no Windows, ou Terminal no Mac/Linux) dentro dessa pasta.
3. Instale as dependências:

```bash
pip install -r requirements.txt
```

## 3. Criando o bot no Telegram

1. No Telegram, converse com **@BotFather**.
2. Envie `/newbot` e siga as instruções (nome e username do bot).
3. O BotFather vai te dar um **token** (algo como `123456789:ABCdefGhIJKlmNoPQRstuVWxyz`). Guarde esse token.
4. Vá até o seu **canal do Telegram** > Administradores > Adicionar Administrador > adicione o bot que você acabou de criar, com permissão de **postar mensagens**.
5. Para descobrir o ID do canal:
   - Se o canal é público, use `@nome_do_canal` diretamente.
   - Se é privado, encaminhe uma mensagem do canal para o bot `@userinfobot` para descobrir o ID numérico (formato `-100xxxxxxxxxx`).

## 4. Pegando as credenciais da Shopee

1. Acesse o [Shopee Affiliate Open Platform](https://affiliate.shopee.com.br) (ou o domínio do seu país) e entre com sua conta de afiliado.
2. Na área de desenvolvedor/API, gere/consulte seu `App ID` e `Secret`.
3. Anote também seu `ID de afiliado` (Affiliate ID), usado para os links de comissão.

> ⚠️ A API de Afiliados da Shopee exige aprovação prévia da conta e pode variar
> ligeiramente conforme o país (domínio da API, campos disponíveis). Se o
> `SHOPEE_API_URL` padrão não funcionar, verifique na documentação oficial da
> sua região o endpoint correto.

## 5. Configurando o arquivo `.env`

1. Renomeie o arquivo `.env.example` para `.env`.
2. Abra o `.env` em um editor de texto e preencha todos os campos:
   - `SHOPEE_APP_ID`, `SHOPEE_SECRET`, `SHOPEE_AFFILIATE_ID`
   - `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHANNEL_ID`
   - `TELEGRAM_CHANNEL_NAME`, `TELEGRAM_CHANNEL_LINK`
3. Salve o arquivo.

**Nunca compartilhe o arquivo `.env` com ninguém** — ele contém suas chaves de acesso.

## 6. Rodando o bot

Postar uma rodada de produtos agora:

```bash
python main.py
```

Rodar continuamente, postando a cada X segundos (definido em `POST_INTERVAL_SEGUNDOS` no `.env`, padrão 30 segundos):

```bash
python main.py --loop
```

> ⚠️ **Atenção com o `SHOPEE_PRODUCT_LIMIT`**: se ele estiver maior que 1, cada
> rodada posta vários produtos seguidos (com 3s de intervalo entre eles) e só
> depois espera os `POST_INTERVAL_SEGUNDOS`. Se você quer **exatamente 1 post
> a cada 30 segundos**, deixe `SHOPEE_PRODUCT_LIMIT=1` (é o padrão).
>
> ⚠️ **Limite do Telegram**: postar rápido demais no mesmo canal pode fazer o
> Telegram limitar temporariamente o bot (flood control). Um post a cada 30s
> é seguro; evite baixar muito esse intervalo.
>
> ⚠️ **Limite da API da Shopee**: chamar a API a cada 30 segundos, 24h por
> dia, pode esbarrar em limites de requisição da sua conta. Se começar a
> tomar erro de "too many requests", aumente o `POST_INTERVAL_SEGUNDOS`.

## 8. Por que ele não repete o mesmo produto

O script guarda, num arquivo local chamado `produtos_postados.json` (o nome é
configurável em `ARQUIVO_POSTADOS`), os links de todos os produtos que já
foram postados. A cada rodada, ele busca os mais vendidos, tira os que já
estão nesse arquivo, e só posta produto **novo**.

Isso significa:

- Se em algum momento aparecer no terminal a mensagem "Todos os produtos
  encontrados já foram postados antes", é porque a lista de mais vendidos não
  mudou desde a última rodada. Duas soluções:
  - Aumente `SHOPEE_BUSCA_BRUTA` no `.env` (ex: de 50 para 150), pra puxar
    mais produtos da Shopee e ter mais opções novas pra escolher.
  - Ou espere: a lista de mais vendidos muda ao longo do dia.
- **Não apague o `produtos_postados.json`** a menos que você queira postar
  tudo de novo do zero (ex: se resetou o canal).
- Se você mudar de computador/servidor, leve esse arquivo junto, senão o bot
  vai postar tudo de novo achando que é a primeira vez.

Deixe esse comando rodando em um servidor, VPS, ou computador ligado. Para
manter rodando em segundo plano em um servidor Linux, você pode usar, por
exemplo:

```bash
nohup python main.py --loop &
```

## 7. Personalizando a busca de produtos

No `.env`:

- `SHOPEE_SEARCH_KEYWORD`: filtra por palavra-chave (ex: `fone de ouvido`). Deixe vazio para trazer ofertas gerais.
- `SHOPEE_PRODUCT_LIMIT`: quantos produtos postar por rodada, depois do filtro.
- `SHOPEE_VENDAS_MINIMAS`: só posta produtos com mais vendas que esse número (padrão: 1000).
- `SHOPEE_AVALIACAO_MINIMA`: só posta produtos com avaliação igual ou maior (padrão: 4.5).
- `SHOPEE_BUSCA_BRUTA`: quantos produtos pedir pra API antes de aplicar os filtros acima. A API não filtra por vendas/avaliação diretamente, então o script busca uma lista maior (ordenada por mais vendidos) e filtra localmente. Se estiver vindo pouco produto depois do filtro, aumente esse número.

O preço mostrado é único (não mais faixa mín-máx) e formatado no padrão brasileiro, ex: `R$ 46,69`.

## Estrutura dos arquivos

```
shopee_telegram_bot/
├── main.py            # script principal
├── requirements.txt   # dependências Python
├── .env.example        # modelo de configuração (renomeie para .env)
└── README.md           # este arquivo
```


## Render (Web Service)

O bot abre um servidor HTTP simples na porta definida pela variável `PORT` do Render (padrão local: `10000`). O endpoint `/` responde `200 OK` para health checks e UptimeRobot.

Start Command:

```bash
python main.py --loop
```
