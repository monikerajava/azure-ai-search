
---

# Construindo um Índice de Pesquisa no Azure AI Search para Análise de Avaliações de Clientes

Este documento detalha os passos práticos para criar uma solução de *knowledge mining* utilizando o Azure AI Search, focando na indexação e análise de avaliações de clientes. O cenário baseia-se em uma cadeia de cafeteria hipotética, a Fourth Coffee, buscando insights sobre a experiência do cliente a partir de avaliações.

A solução demonstra como extrair dados de uma fonte de dados, enriquecê-los com habilidades de IA e torná-los pesquisáveis através de um índice.

## 1. Recursos Essenciais do Azure

Para implementar esta solução, são necessários os seguintes recursos no Azure:

*   **Azure AI Search**: Gerencia a indexação e as consultas.
*   **Azure AI services**: Fornece habilidades de IA para enriquecer os dados. **Importante**: Ambos os recursos (Azure AI Search e Azure AI services) **devem estar na mesma localização (região)**.
*   **Storage account**: Armazena os documentos brutos e outras coleções de dados.

## 2. Criação dos Recursos do Azure

Os recursos são provisionados através do portal do Azure.

### Criar um Recurso Azure AI Search

1.  Acesse o portal do Azure e clique em "+ Create a resource".
2.  Procure por "Azure AI Search" e crie um recurso com as seguintes configurações:
    *   **Subscription**: Sua assinatura Azure.
    *   **Resource group**: Selecione ou crie um grupo de recursos com um nome único.
    *   **Service name**: Um nome único.
    *   **Location**: Escolha qualquer região disponível.
    *   **Pricing tier**: **Basic**.
3.  Selecione "Review + create" e depois "Create" após a validação ser bem-sucedida.
4.  Após a implantação, vá para o recurso.

### Criar um Recurso Azure AI services

1.  Retorne à página inicial do portal do Azure e clique em "+ Create a resource".
2.  Procure por "Azure AI services" e selecione "create" para criar um plano.
3.  Configure-o com as seguintes configurações:
    *   **Subscription**: Sua assinatura Azure.
    *   **Resource group**: **O mesmo grupo de recursos** do seu Azure AI Search.
    *   **Region**: **A mesma localização** do seu Azure AI Search.
    *   **Name**: Um nome único.
    *   **Pricing tier**: **Standard S0**.
    *   **Termos**: Marque a caixa de aceitação.
4.  Selecione "Review + create" e depois "Create" após a validação ser bem-sucedida. Aguarde a conclusão da implantação.

### Criar uma Conta de Armazenamento (Storage Account)

1.  Retorne à página inicial do portal do Azure e clique em "+ Create a resource".
2.  Procure por "storage account" e crie um recurso com as seguintes configurações:
    *   **Subscription**: Sua assinatura Azure.
    *   **Resource group**: **O mesmo grupo de recursos** dos seus recursos Azure AI Search e Azure AI services.
    *   **Storage account name**: Um nome único.
    *   **Location**: Escolha qualquer localização disponível.
    *   **Performance**: Standard.
    *   **Redundancy**: Locally redundant storage (LRS).
3.  Clique em "Review" e depois em "Create". Aguarde a conclusão da implantação e vá para o recurso.
4.  No menu esquerdo da conta de armazenamento, selecione "Configuration".
5.  Altere a configuração para "Allow Blob anonymous access" para **Enabled** e selecione "Save".

## 3. Upload dos Documentos para o Azure Storage

1.  No menu esquerdo da conta de armazenamento, selecione "Containers".
2.  Selecione "+ Container" e configure com as seguintes opções:
    *   **Name**: `coffee-reviews`.
    *   **Public access level**: Container (anonymous read access for containers and blobs).
3.  Em uma nova aba do navegador, baixe o arquivo zip das avaliações de café em **`https://aka.ms/mslearn-coffee-reviews`** e extraia os arquivos para a pasta `reviews`.
4.  No portal do Azure, selecione o container `coffee-reviews`.
5.  Dentro do container, selecione "Upload". No painel "Upload blob", selecione "Select a file", escolha **todos** os arquivos na pasta `reviews`, selecione "Open" e depois "Upload".

## 4. Indexação dos Documentos

Após ter os documentos no armazenamento, utilize o assistente "Import data wizard" no Azure AI Search para criar automaticamente um índice e um indexador.

1.  Navegue até o recurso Azure AI Search no portal do Azure. Na página "Overview", selecione **"Import data"**.
2.  Na página "Connect to your data", em "Data Source", selecione **"Azure Blob Storage"**.
3.  Complete os detalhes da fonte de dados:
    *   **Data Source**: Azure Blob Storage.
    *   **Data source name**: `coffee-customer-data`.
    *   **Data to extract**: Content and metadata.
    *   **Parsing mode**: Default.
    *   **Connection string**: Selecione "Choose an existing connection", escolha sua conta de armazenamento, selecione o container `coffee-reviews` e clique em "Select".
    *   **Container name**: Auto-preenchido.
    *   **Blob folder**: Deixe em branco.
    *   **Description**: Reviews for Fourth Coffee shops.
4.  Selecione **"Next: Add cognitive skills (Optional)"**.
5.  Em "Attach AI Services", selecione seu recurso Azure AI services.
6.  Em "Add enrichments":
    *   Mude o **Skillset name** para `coffee-skillset`.
    *   Selecione a caixa **"Enable OCR and merge all text into merged_content field"**. **Nota**: Habilitar OCR é importante para ver todas as opções de campos enriquecidos.
    *   Verifique se o **Source data field** está definido como `merged_content`.
    *   Altere o **Enrichment granularity level** para **Pages (5000 character chunks)**.
    *   Não selecione "Enable incremental enrichment".
7.  Selecione os seguintes campos enriquecidos:
    *   Extract location names (campo: `locations`).
    *   Extract key phrases (campo: `keyphrases`).
    *   Detect sentiment (campo: `sentiment`).
    *   Generate tags from images (campo: `imageTags`).
    *   Generate captions from images (campo: `imageCaption`).
8.  Sob "Save enrichments to a knowledge store", selecione as seguintes opções:
    *   Image projections.
    *   Documents.
    *   Pages.
    *   Key phrases.
    *   Entities.
    *   Image details.
    *   Image references.
    *   Uma advertência pedindo uma **Storage Account Connection String** aparecerá. Selecione "Choose an existing connection", escolha a conta de armazenamento criada anteriormente.
    *   Clique em "+ Container" para criar um novo container chamado **`knowledge-store`** com o nível de privacidade **Private**, e selecione "Create".
    *   Selecione o container `knowledge-store` e clique em "Select".
    *   Selecione "Azure blob projections: Document". O container `knowledge-store` deve ser auto-preenchido.
9.  Selecione **"Next: Customize target index"**.
10. Mude o **Index name** para **`coffee-index`**.
11. Certifique-se de que a **Key** está definida como `metadata_storage_path`. Deixe "Suggester name" em branco e "Search mode" auto-preenchido.
12. Revise as configurações padrão dos campos do índice. **Selecione "filterable"** para todos os campos que já estão selecionados por padrão, incluindo: `content`, `locations`, `keyphrases`, `sentiment`, `merged_content`, `text`, `layoutText`, `imageTags`, `imageCaption`.
13. Selecione **"Next: Create an indexer"**.
14. Mude o **Indexer name** para **`coffee-indexer`**.
15. Deixe o **Schedule** definido como **Once**.
16. Expanda "Advanced options" e certifique-se de que **"Base-64 Encode Keys"** está selecionado para tornar o índice mais eficiente.
17. Selecione **"Submit"** para criar a fonte de dados, o skillset, o índice e o indexador. O indexador será executado automaticamente, executando o pipeline de indexação:
    *   Extrai campos de metadados e conteúdo da fonte de dados.
    *   Executa o skillset de habilidades cognitivas para gerar campos enriquecidos.
    *   Mapeia os campos extraídos para o índice.

## 5. Monitoramento do Indexador

1.  Retorne à página do seu recurso Azure AI Search. No painel esquerdo, em "Search Management", selecione "Indexers".
2.  Selecione o `coffee-indexer` recém-criado.
3.  Aguarde um minuto e selecione **"&orarr; Refresh"** até que o **Status** indique sucesso.
4.  Selecione o nome do indexador para ver mais detalhes.

## 6. Consulta do Índice

Utilize o Search explorer para escrever e testar consultas.

1.  Na página "Overview" do seu serviço de pesquisa, selecione **"Search explorer"** no topo da tela.
2.  Verifique se o índice selecionado é o `coffee-index`. Abaixo do índice selecionado, mude a **view** para **JSON view**.
3.  No campo **JSON query editor**, copie e cole o seguinte para retornar todos os documentos e a contagem:
    ```json
    { "search": "*", "count": true }
    ```
    Selecione **"Search"**. O resultado incluirá um campo `@odata.count` com o total de documentos.
4.  Para filtrar por localização, copie e cole:
    ```json
    { "search": "locations:'Chicago'", "count": true }
    ```
    Selecione **"Search"**. A consulta filtrará por avaliações com a localização "Chicago".
5.  Para filtrar por sentimento, copie e cole:
    ```json
    { "search": "sentiment:'negative'", "count": true }
    ```
    Selecione **"Search"**. A consulta filtrará por avaliações com sentimento "negative".
    **Nota**: Os resultados são ordenados por `@search.score`, que indica quão bem correspondem à consulta.
6.  O Search explorer permite explorar os resultados para entender, por exemplo, as key phrases associadas a avaliações negativas.

## 7. Revisão do Knowledge Store

O *Import data wizard* também cria um knowledge store, onde os dados enriquecidos pelas habilidades de IA são persistidos como projeções e tabelas.

1.  Navegue de volta para sua conta de armazenamento do Azure no portal.
2.  No menu esquerdo, selecione "Containers". Selecione o container `knowledge-store`.
3.  Você verá pastas para metadados de cada documento. Selecione qualquer pasta. Dentro da pasta, clique no arquivo **`objectprojection.json`**.
4.  Selecione **"Edit"** para ver o JSON produzido para um dos documentos.
5.  Retorne à lista de containers e selecione o container `coffee-skillset-image-projection`. Selecione qualquer item, e então selecione qualquer arquivo **`.jpg`**.
6.  Selecione **"Edit"** para ver a imagem armazenada do documento. Todas as imagens são armazenadas desta maneira.
7.  Retorne à conta de armazenamento e selecione **"Storage browser"** no painel esquerdo, e depois **"Tables"**.
8.  Há uma tabela para cada entidade no índice. Selecione a tabela **`coffeeSkillsetKeyPhrases`**.
9.  Examine as *key phrases* capturadas do conteúdo das avaliações. Muitos campos são chaves que permitem vincular tabelas como em um banco de dados relacional. O último campo mostra as *key phrases* extraídas.

## 8. Próximos Passos

Este exemplo simples demonstra apenas algumas das capacidades do Azure AI Search. Para aprender mais, consulte a página do serviço **Azure AI Search**.

---

Este resumo detalhado abrange os principais componentes e passos da criação de uma solução de pesquisa e *knowledge mining* no Azure, desde o provisionamento de recursos até a indexação, enriquecimento e exploração dos dados. Espero que seja útil para seus objetivos!

## Links úteis
Documentação de referência

https://aka.ms/ai900-ai-search


Explore an Azure AI Search index (UI) - Laboratório no Microsoft Learning

https://microsoftlearning.github.io/mslearn-ai-fundamentals/Instructions/Labs/11-ai-search.html

