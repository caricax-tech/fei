## **Modelo 1: O Pipeline AI-Powered (No/Low-Code) – "A Abordagem do Curador"**

Neste paradigma, o desenvolvedor atua menos como um construtor e mais como um curador ou orquestrador de serviços gerenciados. A IA não apenas assiste, mas executa porções significativas da criação. O objetivo é a máxima velocidade de prototipagem com o mínimo de código manuscrito.

**Filosofia:** A velocidade supera a customização. O foco está no "o quê" (o resultado final) e não no "como" (a implementação detalhada). O sistema é construído sobre plataformas como serviço (PaaS) que abstraem a infraestrutura.

**Componentes Chave:**
*   **Back-end/Infraestrutura:** Firebase (Firestore, Cloud Functions, Hosting, Authentication).
*   **Motor Cognitivo:** Gemini API (acessada via Cloud Function).
*   **Ferramenta de Geração:** Gemini CLI ou Google AI Studio para prototipar a lógica de IA.
*   **Front-end:** Uma aplicação web simples, possivelmente em HTML/CSS/JS puros ou um framework leve como Vue.js/Svelte.

#### **Pipeline de Execução:**

**Fase 1: Fundação da Infraestrutura (5 minutos)**
1.  Crie um novo projeto no console do Firebase.
2.  Ative os serviços: Firestore (banco de dados NoSQL), Cloud Functions e Hosting.
3.  Instale o Firebase CLI localmente: `npm install -g firebase-tools`.
4.  Faça login (`firebase login`) e inicialize o projeto no seu diretório local (`firebase init`). Selecione Firestore, Functions e Hosting. O Firebase CLI criará toda a estrutura de arquivos necessária.

**Fase 2: Modelo de Dados e Ingestão (No-Code)**
1.  No console do Firebase, crie manualmente duas coleções no Firestore:
    *   `raw_documents`: Para armazenar o texto bruto a ser analisado. Cada documento pode ter um campo `text`.
    *   `generated_graphs`: Para armazenar a saída estruturada da IA.
2.  Adicione um documento de teste na coleção `raw_documents` com algum texto para iniciar.

**Fase 3: O Núcleo Cognitivo (Low-Code)**
1.  **Prototipagem do Prompt (AI-Powered):** Use o Gemini CLI ou o Google AI Studio para refinar o prompt que transforma o texto em um grafo.
    *   **Exemplo de Prompt para o Gemini:**
        ```
        Contexto: Você é um sistema de extração de conhecimento. Analise o texto fornecido e retorne uma estrutura JSON contendo "nodes" (entidades) e "edges" (relações).

        Texto: "O projeto Alfa, liderado por Maria, usa a API da Apollo para se conectar ao sistema de inventário. Carlos, da equipe de QA, reportou um bug na integração."

        Regras de Saída:
        - O JSON deve ter duas chaves de nível superior: "nodes" e "edges".
        - Cada nó deve ter "id" (string, lowercase, sem espaços), "label" (string original) e "type" (e.g., 'Person', 'Project', 'API', 'System').
        - Cada aresta deve ter "source" (id do nó de origem), "target" (id do nó de destino) e "label" (a relação, em uma palavra).

        Retorne APENAS o JSON.
        ```
2.  **Implementação da Cloud Function:** Navegue até o diretório `functions` criado pelo Firebase CLI.
3.  Edite o arquivo `index.js` (ou `.ts`) para criar uma função que é acionada sempre que um novo documento é adicionado a `raw_documents`.
    *   **Código da Cloud Function (gerado com assistência de IA):**
        ```javascript
        const functions = require("firebase-functions");
        const admin = require("firebase-admin");
        const { GoogleAuth } = require("google-auth-library");
        const { DiscussServiceClient } = require("@google-ai/generativelanguage");

        admin.initializeApp();
        const firestore = admin.firestore();

        const MODEL_NAME = "models/text-bison-001"; // ou outro modelo Gemini
        const API_KEY = functions.config().gemini.key; // Armazene a chave de forma segura
        const client = new DiscussServiceClient({
          authClient: new GoogleAuth().fromAPIKey(API_KEY),
        });

        exports.generateGraphFromText = functions.firestore
          .document("raw_documents/{docId}")
          .onCreate(async (snap, context) => {
            const rawData = snap.data();
            const textToAnalyze = rawData.text;

            // O prompt que foi prototipado na etapa anterior
            const prompt = `... seu prompt aqui ... \nTexto: "${textToAnalyze}"`;

            const result = await client.generateText({
              model: MODEL_NAME,
              prompt: { text: prompt },
            });
            
            // Assume-se que a IA retorna um JSON stringificado
            const graphData = JSON.parse(result[0].candidates[0].output);

            // Salva o grafo gerado na outra coleção
            return firestore.collection("generated_graphs").doc(context.params.docId).set(graphData);
          });
        ```

**Fase 4: Visualização no Front-end (Low-Code)**
1.  No diretório `public` (ou o diretório de hosting que você configurou), crie um `index.html`.
2.  Use uma biblioteca de grafos como `Vis.js` ou `Cytoscape.js` via CDN para simplificar.
3.  Escreva um script simples para se conectar ao Firestore (usando o SDK do Firebase para a web), escutar as mudanças na coleção `generated_graphs` e renderizar os dados.

**Implicações:** Este pipeline é extremamente rápido para validar uma ideia. A "inteligência" está totalmente delegada à IA (para gerar o grafo) e à plataforma (para gerenciar a infraestrutura). O desenvolvedor é um integrador. A desvantagem é o aprisionamento tecnológico (vendor lock-in) e a menor flexibilidade para otimizações de baixo nível.

---

### **Modelo 2: O Pipeline AI-Assisting (Real Code) – "A Forja do Demiurgo"**

Aqui, o desenvolvedor está no controle total, usando a IA como um assistente de programação onipresente e altamente capacitado. A arquitetura é robusta, customizável e segue as melhores práticas de engenharia de software.

**Filosofia:** O controle e a customização são primordiais. A IA acelera o "como", mas o desenvolvedor define a arquitetura. O código é o ativo principal.

**Componentes Chave:**
*   **Back-end:** Node.js + Express (ou Python + FastAPI) em um contêiner Docker.
*   **Front-end:** Angular (ou React) com CLI e ferramental completo.
*   **Banco de Dados:** PostgreSQL (rodando em Docker para desenvolvimento, ou um serviço gerenciado como o Cloud SQL).
*   **Motor Cognitivo:** SDK oficial do Google para a API Gemini (e.g., `@google/generative-ai`).
*   **Ferramenta de Assistência:** GitHub Copilot / Gemini Code Assist integrado ao VS Code.

#### **Pipeline de Execução:**

**Fase 1: Arquitetura e Estrutura (AI-Assisted)**
1.  **AI-Assist:** "Crie uma estrutura de monorepo usando Nx com um aplicativo Angular chamado 'graph-viewer' e um aplicativo de back-end Node/Express chamado 'api'."
2.  **Desenvolvedor:** Revisa e ajusta a estrutura gerada. Usa o Angular CLI (`ng g component graph-view`) e geradores do Express para criar a estrutura básica de componentes, serviços e rotas.
3.  **AI-Assist:** "Escreva um `docker-compose.yml` que configure os serviços 'api', 'graph-viewer' e um banco de dados PostgreSQL."
4.  **Desenvolvedor:** Executa `docker-compose up` para ter todo o ambiente de desenvolvimento rodando localmente.

**Fase 2: Desenvolvimento do Back-end (AI-Assisted)**
1.  **Desenvolvedor:** Define as rotas da API no Express: `POST /documents` (para receber texto) e `GET /graphs/:docId` (para retornar o grafo processado).
2.  **AI-Assist:** "Crie um serviço em TypeScript que, ao receber um texto, usa o SDK `@google/generative-ai` para chamar o modelo Gemini com o seguinte prompt [insira o prompt refinado]. O serviço deve validar a resposta JSON da IA, tratar erros e persistir os nós e arestas em tabelas 'nodes' e 'edges' do PostgreSQL usando o ORM Prisma."
3.  **Desenvolvedor:** Revisa o código gerado, ajusta os modelos do Prisma, refina o tratamento de erros (e.g., retentativas com backoff exponencial) e escreve testes de unidade. A IA ajuda a gerar os casos de teste.

**Fase 3: Desenvolvimento do Front-end (AI-Assisted)**
1.  **Desenvolvedor:** No componente Angular `graph-view`, planeja a interação: o componente deve buscar os dados do grafo da API do back-end e renderizá-los.
2.  **AI-Assist:** "Crie um serviço Angular chamado `GraphApiService` com um método `getGraph(docId)` que faz uma chamada GET para a rota `/api/graphs/:docId`."
3.  **Desenvolvedor:** No `graph-view.component.ts`, injeta o `GraphApiService` e chama o método.
4.  **AI-Assist:** "Usando a biblioteca Cytoscape.js, escreva o código para inicializar um grafo no `ngAfterViewInit` do meu componente Angular, usando os dados recebidos do `GraphApiService`. Configure o estilo dos nós e arestas com uma estética minimalista e profissional."
5.  **Desenvolvedor:** Refina o estilo, adiciona interatividade (e.g., painel de detalhes ao clicar em um nó) e garante que o gerenciamento de estado seja robusto (usando RxJS ou Signals).

**Fase 4: Pipeline de CI/CD e Implantação (AI-Assisted)**
1.  **AI-Assist:** "Escreva um Dockerfile para produção para a API Node.js, usando um build multi-stage."
2.  **AI-Assist:** "Crie um workflow do GitHub Actions que, a cada push para a branch `main`, constrói as imagens Docker da API e do front-end, as envia para o Google Artifact Registry e as implanta no Google Cloud Run."
3.  **Desenvolvedor:** Configura os segredos (chaves de API, credenciais de nuvem) no GitHub e ajusta o workflow para as suas necessidades específicas.

**Implicações:** Este pipeline resulta em um produto de nível profissional, escalável e de fácil manutenção. O desenvolvedor mantém o controle total sobre a arquitetura e a tecnologia, usando a IA para anular o trabalho repetitivo e acelerar a implementação. O custo é um tempo de desenvolvimento inicial maior e uma complexidade de gerenciamento mais elevada, o que é uma troca deliberada em favor do poder e da flexibilidade.

**Conclusão**

Ambos os pipelines atingem o mesmo objetivo funcional, mas representam um deslocamento epistemológico na forma como o software é criado. O primeiro é um ato de **curadoria**, o segundo, um ato de **construção assistida**. A escolha entre eles não é técnica, mas estratégica, dependendo do contexto do projeto: velocidade de validação versus robustez de longo prazo.
