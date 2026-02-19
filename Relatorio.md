# Relátorio de pesquisa

Victor Carneiro dos Santos Angelo - 122110725

## Geração do Dataset

### Metodologia de Geração

O processo de construção do dataset seguiu o fluxo abaixo:

1. **Extração de Dados:** Coleta de informações contextuais da Wikipedia sobre os clubes da Série A e o histórico da competição.
2. **Geração Sintética:** Utilização do modelo **Gemini 3 Flash** para gerar 1.000 instruções, seguindo o formato de dados proposto no repositório *LLMs from Scratch* (Sebastian Raschka).
3. **Auditoria e Curadoria:** O modelo **Gemini 3 Pro** foi empregado para analisar o lote gerado, com o objetivo de excluir:
    * **Vazamento de Resposta (Data Leakage):** Instruções onde a resposta já estava contida ou sugerida no campo de entrada (`input`).
    * **Instruções Inválidas:** Exemplos com erros lógicos ou falta de utilidade prática.

### Primeira tentativa

1. **Base de Conhecimento:** Foram utilizadas informações extraídas de páginas da Wikipédia abrangendo todos os clubes da Série A do Brasileirão, além de dados sobre competições sul-americanas e seleções nacionais.

2. **Disponibilidade:** O dataset completo pode ser acessado através do seguinte link: [Google Drive - Dataset](https://drive.google.com/file/d/1Blp1UxFdwOGqys8hihqVbEYZ8XB4UvMm/view?usp=sharing).

3. **Estatísticas e Filtragem:** Inicialmente, foram geradas **1.000 instruções**. Após uma etapa de filtragem (para remoção de inconsistências e vazamento de dados), restaram **930 instruções** válidas. Exemplos de entradas descartadas:  Foram removidos casos de *data leakage* (onde a resposta estava no input) ou contradições factuais no enunciado.

   ```json
   [
     {
       "instruction": "Name the stadium that hosted the opening match of the 2014 World Cup.",
       "input": "Arena Corinthians",
       "output": "The match was held at the Arena Corinthians (now Neo Química Arena) in São Paulo."
     },
     {
       "instruction": "Identify the city where the 'Arena Pantanal' is located.",
       "input": "Curitiba",
       "output": "The stadium is located in Cuiabá."
     }
   ]
    ```

4. **Problemas:** durante a fase de testes, identificou-se uma acentuada dificuldade do modelo GPT-2 em responder corretamente às perguntas. Hipótese Principal: A baixa performance e as alucinações sugerem que o modelo original não foi exposto a esses dados específicos durante o seu pré-treino. Como o GPT-2 carece dessa base de conhecimento prévia sobre o nicho do futebol sul-americano, o fine-tuning de instruções não foi suficiente para garantir a precisão factual.

### Segunda tentativa

1. **Base de Conhecimento:** Com o objetivo de mitigar as dificuldades de convergência e alucinação da primeira tentativa, o foco da extração de dados foi alterado. Substituímos o nicho específico do futebol por páginas da Wikipédia com temas de conhecimento geral, incluindo literatura clássica, obras populares e fatos históricos amplamente documentados.

2. **Disponibilidade:** O dataset completo pode ser acessado através do seguinte link: [Google Drive - Dataset](https://drive.google.com/file/d/1YoNa2JeW51IbycYprwoQCxm1lIhOf6-T/view?usp=sharing).

3. **Estatísticas e Filtragem:** Inicialmente, foram geradas **1.100 instruções**. Após uma etapa de filtragem (para remoção de inconsistências e vazamento de dados), restaram **1010 instruções** válidas.

### Prompts utilizados

1. **Geração de Dados**

    ```txt
    ===
    Tarefa: Agir como um gerador de datasets sintéticos para fine-tuning de modelos de linguagem.

    ===
    Objetivo: Gerar pares de Instrução, Entrada e Saída seguindo rigorosamente as diretrizes abaixo.
    
    ===
    Regras de Geração
        Idioma: Todo o conteúdo deve ser escrito em Inglês.
        Base de Conhecimento: Utilize apenas informações de 2022 ou anteriores. Priorize fatos de conhecimento público presentes nas páginas da wikipedia fornecidas.
        Uso do Campo 'Input': Utilize o campo input apenas se a instrução exigir um contexto ou dado externo para ser completada. Caso contrário, preencha com "".
        Formato de Saída: Retorne os dados estritamente em formato JSON (lista de objetos).

    ===
    Exemplo:
        {
            "instruction": "Evaluate the following phrase by transforming it into the spelling given.",
            "input": "freind --> friend",
            "output": "The spelling of the given phrase \"freind\" is corrected to \"friend\"."
        }
    ```

2. **Filtragem de Dados**

   ```txt
    ===
    Você é um Especialista em Curadoria de Dados (Data Curator) para modelos de linguagem. Sua missão é analisar um arquivo JSON contendo pares de instrução/input/output e filtrar apenas os dados que cumprem 100% dos critérios de qualidade. como um gerador de datasets sintéticos para fine-tuning de modelos de linguagem.

    ===
    Critérios de Avaliação (Filtros)

    Corte Temporal (Crucial): Remova qualquer exemplo que mencione eventos, tecnologias ou fatos ocorridos após 31 de dezembro de 2022. (Ex: Lançamento do GPT-4, eventos políticos de 2023/2024, etc).
    Idioma: Todas as instruções, entradas e saídas devem estar obrigatoriamente em Inglês.
    Integridade do Formato JSON: Verifique se o campo input está vazio ("") quando a instrução for autossuficiente e se os campos instruction e output estão presentes e coerentes entre si.
    Veracidade e Fonte: O conteúdo deve ser baseado em conhecimento público e factual (estilo Wikipédia). Remova alucinações ou informações falsas.
    Qualidade da Resposta: A resposta no campo output deve ser útil, direta e gramaticalmente correta.

    ===
    DADOS
    Arquivo Json
    ```

## Treinamento

### Dataset

Para melhorar o dataset criado, foi realizado a concatenção com o dataset original do repositório do Sebastian Raschka. Foram utilizadas 1000 instruções de cada, totalizando 2000 instruções.

### Configuração do Experimento (Hiperparâmetros)

O treinamento foi executado utilizando o script oficial do repositório *LLMs from Scratch*, configurado com os seguintes hiperparâmetros:

* **Batch Size:** 8
* **Learning Rate (LR):** 5e-5 (0.00005)
* **Weight Decay:** 0.1
* **Épocas:** 1

### Modelos e Divisão de Dados

Foram conduzidos dois experimentos distintos utilizando a arquitetura **GPT-2 (pré-treinado)**:

1. **Modelo GPT-2 Full:** Treinado com o dataset completo de 2.000 instruções.
    * *Split:* 1.600 (Treino) | 200 (Validação) | 200 (Teste).
2. **Modelo GPT-2 Small:** Treinado com um subconjunto reduzido de 500 instruções.
    * *Split:* 400 (Treino) | 50 (Validação) | 50 (Teste).

### Resultados e Avaliação

Os modelos foram avaliados com base na perda de validação (*Validation Loss*), apresentando os seguintes indicadores:

| Modelo | Val Loss |
| :--- | :--- |
| **GPT-2_Full** | 0.504 |
| **GPT-2_Small** | 0.596 |

**1. Modelo GPT-2 Full (2.000 instruções)**

![Curva de Loss - Full Dataset](https://github.com/VictorCSA/InstructionTuning-GPT-2-/blob/main/imagens/loss-plot_full.png?raw=true)

**2. Modelo GPT-2 Small (500 instruções)**

![Curva de Loss - Small Dataset](https://github.com/VictorCSA/InstructionTuning-GPT-2-/blob/main/imagens/loss-plot_small.png?raw=true)

## Testes e Avaliação Qualitativa

### Geração de Respostas Estruturadas (JSON)

Para comparar o desempenho dos modelos, foram gerados dois arquivos JSON contendo as inferências baseadas em instruções de teste. O objetivo foi contrastar a capacidade de resposta entre o modelo original e as variantes ajustadas.

1. **Comparativo Base vs. Full:** Contém as respostas do modelo original (sem *fine-tuning*) e do modelo ajustado com o dataset de 2.000 instruções.
2. **Modelo Small:** Contém as inferências do modelo ajustado com apenas 500 instruções.

#### Exemplo de Estrutura de Saída

O exemplo abaixo ilustra o comportamento típico de cada modelo. Note que o modelo sem o ajuste adequado tende a entrar em um *loop* de repetição do prompt (regressão infinita), enquanto o modelo ajustado busca formular uma definição direta.

```json
{
  "instruction": "Define the term 'kinetic energy'.",
  "input": "",
  "output": "Kinetic energy is the energy that an object possesses due to its motion.",
  "model_1_response": "Kinetic energy is the energy that is released by a body as a result of physical activity.",
  "model_2_response": "Write a response that appropriately completes the request.\n\n### Instruction:\n\nDefine the term 'kinetic energy'.\n\n\n\nWrite a response that appropriately completes the request. [REPETIÇÃO]"
}
```

## Avaliação: LLM as a Judge

Para uma análise comparativa e quantitativa das inferências, implementamos a metodologia **LLM as a Judge**. Utilizou-se uma instância local do modelo **Llama 7B** como juiz imparcial para avaliar as respostas geradas pelos modelos ajustados.

### Metodologia de Avaliação

O juiz avaliou cada par de respostas (Modelo A vs. Modelo B) de forma cega, baseando-se em três pilares fundamentais, com pontuações de 0 a 5:

1. **Acurácia Factual (*Factual Correctness*):** Veracidade das informações fornecidas.
2. **Aderência à Instrução (*Instruction Adherence*):** Capacidade do modelo de seguir exatamente o que foi solicitado.
3. **Clareza e Utilidade (*Clarity and Usefulness*):** Qualidade da escrita e quão útil a resposta é para o usuário final.

### Configuração do Prompt de Avaliação

O modelo avaliador foi instruído a retornar estritamente um objeto JSON, garantindo que os dados pudessem ser processados programaticamente para a geração de métricas finais.

```python
# Estrutura de saída esperada (Strict JSON)
example_output = {
    "model_a": {
        "factual_correctness": 0,
        "instruction_adherence": 0,
        "clarity_and_usefulness": 0,
        "total": 0
    },
    "model_b": {
        "factual_correctness": 0,
        "instruction_adherence": 0,
        "clarity_and_usefulness": 0,
        "total": 0
    },
    "winner": "A | B | Tie",
    "justification": "Breve comparação objetiva justificando a decisão."
}
```

### Análise Consolidada de Resultados

Os resultados obtidos através da avaliação do LLM as a Judge demonstram um salto qualitativo substancial após o processo de ajuste fino. O modelo GPT-2 Fine-tuned (Full) superou consistentemente a versão base em todas as métricas de performance e utilidade.

#### Comparativo de Performance (Scores Médios)

Abaixo, apresentamos a média detalhada das pontuações atribuídas pelo avaliador (escala 0–5 por critério):

| Métrica | GPT-2 Base (Model B) | GPT-2 Fine-tuned (Model A) |
| :--- | :---: | :---: |
| **Acurácia Factual** | 1.37 | **3.79** |
| **Aderência à Instrução** | 1.54 | **4.12** |
| **Clareza e Utilidade** | 1.48 | **4.15** |
| **Score Total Médio** | 4.39 | **12.06** |

#### Taxa de Vitória (*Win Rate*)

Na comparação direta (*head-to-head*), o modelo ajustado com o dataset completo dominou as avaliações:

* **Vitórias do Modelo A (Full):** 75.00%
* **Vitórias do Modelo B (Base):** 17.50%
* **Empates:** 7.50%

![Distribuição de Scores Totais](https://github.com/VictorCSA/InstructionTuning-GPT-2-/blob/main/imagens/score_percentage.png?raw=true)

#### Impacto do Volume de Dados

A comparação entre as variantes Full (2.000 instruções) e Small (500 instruções) revela que o volume de dados foi determinante para a estabilidade do modelo:

* **GPT-2 Full (Total Médio):** 12.06
* **GPT-2 Small (Total Médio):** 8.46

O ganho médio de performance do modelo Full em relação à base foi de 7.67 pontos, atingindo picos de melhoria de até 15 pontos em instruções complexas onde a base falhava completamente.

#### Resumo Estatístico Percentual

A tabela abaixo normaliza os resultados para uma escala percentual, permitindo visualizar a amplitude de performance de cada configuração:

| Modelo | Score Médio (%) | Melhor Caso (%) | Pior Caso (%) |
| :--- | :---: | :---: | :---: |
| **GPT-2 Base** | 29.25% | 100.0% | 0.00% |
| **GPT-2 Fine-tuned (Small)** | 56.40% | 100.0% | 0.00% |
| **GPT-2 Fine-tuned (Full)** | **80.40%** | 100.0% | 6.67% |

---

