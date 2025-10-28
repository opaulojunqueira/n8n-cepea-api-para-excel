# n8n-cepea-api-para-excel

Este fluxo acessa automaticamente a API do Cepea (ESALQ/USP), extrai as cotações diárias de produtos agropecuários (como algodão, arroz, milho, boi gordo, entre outros), processa o HTML retornado e gera um arquivo Excel (.xlsx) com todas as informações formatadas. Ele foi criado para facilitar a extração periódica de preços e a distribuição das planilhas geradas.

---

### 🧾 Fluxo: CEPEA API para Excel

Este fluxo acessa automaticamente a **API do Cepea (ESALQ/USP)**, extrai as **cotações diárias de produtos agropecuários** (como algodão, arroz, milho, boi gordo, entre outros), **processa o HTML retornado** e **gera um arquivo Excel (.xlsx)** com todas as informações formatadas.

O fluxo pode ser executado manualmente ou agendado (via **Schedule Trigger**).  
Após a execução, o **node `Convert to File`** transforma os dados JSON em uma planilha Excel.

👉 Para baixar o resultado:  
Abra o node **Convert to File** → vá até a aba **Binary Data** → clique em **Download** para obter o arquivo gerado.

---

## Como funciona (visão geral)

1. Trigger
   - Pode ser um gatilho manual (Manual Trigger) para testes ou um **Schedule Trigger** para execução automática (diária, semanal, horário específico).
2. Requisição à API
   - Um node **HTTP Request** faz a chamada à API do Cepea para obter os dados/HTML da cotação do produto e período desejado.
3. Processamento do HTML
   - Nodes como **HTML Extract**, **Function** ou **Function Item** extraem e transformam o HTML em JSON estruturado (colunas: data, produto, local, preço, variação, unidade, observações, etc.).
4. Normalização e limpeza
   - Nodes **Set** / **Function** ajustam nomes de colunas, tipos (datas, números), tratam valores ausentes e padronizam formatos.
5. Geração do Excel
   - Node **Convert to File** (ou similar) converte o JSON em binary .xlsx com cabeçalho, formatação básica (se aplicável) e nome de arquivo configurável.
6. Resultado
   - Arquivo .xlsx disponível para download no node (Binary Data). Opcional: enviar por e-mail, upload para Google Drive/OneDrive/S3 usando nodes correspondentes.

---

## Requisitos

- n8n (versão compatível — recomenda-se a versão mais recente ou ao menos v0.XX+; verifique compatibilidade dos nodes).
- Acesso à API do Cepea (rota pública). Se a API exigir headers ou tokens no futuro, configure no node HTTP Request.
- Nodes nativos do n8n: HTTP Request, HTML Extract (ou equivalente), Function/Set, Convert to File, Schedule Trigger (opcional).
- Conexões externas (opcionais): SMTP, Google Drive, OneDrive, S3, caso deseje armazenar/compartilhar automaticamente.

---

## Configuração recomendada dos nodes

- HTTP Request
  - Método: GET
  - URL: endpoint da API Cepea (ex.: https://www.cepea.esalq.usp.br/… — usar URL de consulta específica)
  - Headers: (se necessário) Content-Type: application/json, Accept: text/html
  - Query Params: produto, data inicial/final, série, etc., conforme a API do Cepea

- HTML Extract / Function
  - Extraia a tabela ou os elementos HTML que contêm as cotações.
  - Transforme linhas de tabela em objetos JSON com chaves consistentes: date, product, local, price, variation, unit

- Function / Set
  - Converter strings numéricas (com vírgula) para número (usar replace(',', '.') e parseFloat).
  - Converter datas para padrão ISO (yyyy-mm-dd) ou para o formato desejado.
  - Remover linhas duplicadas e registros vazios.

- Convert to File
  - Tipo: Excel (.xlsx)
  - Nome do arquivo: ex.: cepea-cotacoes-{{ $now.toISOString().slice(0,10) }}.xlsx
  - Defina se deseja incluir cabeçalho e o nome da planilha (sheetName).
  - Após execução: vá em Binary Data → Download.

---

## Exemplos de uso

- Execução diária automática: agende o Schedule Trigger para rodar todo dia às 07:00, gerar a planilha com as cotações do dia anterior e enviar por e-mail ao time.
- Coleta histórica: rodar manualmente definindo data inicial e final para gerar planilhas com séries históricas.
- Integração com sistemas: após gerar o Excel, subir automaticamente para Google Drive ou enviar via webhook para outra aplicação que consome os dados.

---

## Boas práticas e observações

- Verifique limites e políticas da API do Cepea (rate limits, uso comercial, atribuição).
- Teste com diferentes produtos para garantir que o parser HTML cobre variações na estrutura da página.
- Padronize formatos (datas e números) antes de gerar o .xlsx para evitar células como texto.
- Se necessário, adicione logs e notificação (ex.: enviar e-mail em caso de erro).
- Mantenha a versão do n8n atualizada para evitar incompatibilidades de nodes.

---

## Troubleshooting (problemas comuns)

- Resposta vazia ou erro 404/500 do Cepea: verifique a URL e parâmetros; teste no Postman/cURL.
- Dados em HTML com estrutura diferente: atualize os seletores do HTML Extract ou ajuste a função que parseia o HTML.
- Números com vírgula sendo interpretados como texto no Excel: converta para ponto decimal antes da geração do arquivo.
- Arquivo .xlsx muito grande: considere fragmentar por produto/data ou compactar antes do envio.

---

## Exemplo de workflow (resumido)

1. Schedule Trigger (diário 07:00) (precisa de adicionar)
2. HTTP Request → GET Cepea API (params: produto, inicio, fim)
3. HTML Extract → transformar tabela em JSON
4. Function / Set → normalizar campos
5. Convert to File → gerar cepea-cotacoes-YYYY-MM-DD.xlsx
6. (Opcional) Google Drive upload / E-mail / Webhook

---

## Créditos e contato

- Autor: Paulo Junqueira (opaulojunqueira)
- Site / contato: https://paulojunqueira.com
