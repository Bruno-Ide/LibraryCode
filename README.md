# Estante de códigos

Catálogo de módulos com busca por texto livre. Back-end Flask + SQLite, front-end em HTML/JS puro.

## Rodar localmente (SQLite, sem configurar nada)
    pip install -r requirements.txt
    python app.py          # abre em http://localhost:5000

Na primeira execução o banco `estante.db` é criado a partir de `seed.json` (os 184 módulos da planilha).
Para recomeçar do zero, apague `estante.db`.

## Publicar (Neon = banco, Render = aplicação, ambos no plano gratuito)
1. **Banco:** crie uma conta em neon.com, crie um projeto e copie a *connection string* (`postgresql://...`).
2. **Código:** suba esta pasta para um repositório no GitHub (o `.gitignore` já exclui o banco local).
3. **Aplicação:** em render.com, *New > Blueprint*, escolha o repositório (ele lê o `render.yaml`)
   e cole a connection string da Neon em `DATABASE_URL`.
4. **Pronto:** no primeiro start as tabelas são criadas e os 184 módulos carregados. A URL `*.onrender.com` já tem HTTPS.
5. **Editar:** o `ADMIN_TOKEN` foi gerado pelo Render (aba *Environment*). No site, clique em *Entrar como editor* e cole o token.
   Sem o token, qualquer pessoa pode consultar e buscar, mas não alterar.

Para testar o Postgres no seu computador: `DATABASE_URL="postgresql://..." python app.py`.

**Limites do plano gratuito (verificados em nov/2026):** o Render hiberna a aplicação após 15 min sem acesso
(o primeiro acesso depois disso leva cerca de 1 minuto), e a Neon suspende o banco após 5 min parado
(retoma em milissegundos). Não use o Postgres gratuito do Render: ele expira em 30 dias.

## API
| Método | Rota | O que faz |
|---|---|---|
| GET | `/api/categorias` | categorias com total de módulos |
| GET | `/api/modulos?categoria_id=2` | lista módulos (filtro opcional) |
| GET | `/api/modulos/<id>` | ficha de um módulo |
| POST / PUT / DELETE | `/api/modulos[/<id>]` | cria, edita, exclui (header `Authorization: Bearer <ADMIN_TOKEN>`) |
| POST | `/api/busca` | `{"texto": "...", "categoria_id": 2}` → palavras-chave + módulos ranqueados |

## Como a busca funciona (`app.py`, seção "busca por palavras-chave")
1. Normaliza o texto (minúsculas, sem acentos) e remove palavras vazias ("quero", "preciso", "para"...).
2. Reduz cada palavra ao radical (`pagamentos`, `pagar` → `pag`).
3. Amplia com termos relacionados (`pix` → boleto, cartão, cobrança...), com peso menor. Edite a lista `GRUPOS` para ajustar.
4. Pontua cada módulo: nome pesa mais que categoria/tecnologia, que pesam mais que a descrição; palavras raras valem mais que comuns.

## Ajustes feitos nos dados da planilha
Typos corrigidos (Criptogradia, Baackup, serviçoes, Inbtegração, profisisonais) e a versão do OpenAPI,
que o Excel havia convertido em data (46025), ficou como `3.x`. O módulo 128 não tem "criado por".
