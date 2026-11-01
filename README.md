# agent-julia

Creating a 100% Julia-native agent pipeline with PromptingTools.jl is an excellent choice. The package was designed to be extremely lightweight, high-performing, and expressive, taking full advantage of Julia’s type system and dispatch mechanism.


In PromptingTools.jl, instead of complex and heavy class abstractions, agents are modeled as dispatch functions and typed structs, using function calling and schema extraction mechanisms.



Complete Native Code in Julia
1. Installing Packages
In the Julia REPL (press ] to enter Pkg mode):

pkg> add PromptingTools Pydantic2Julia JSON3 Decimals

2. Implementing the Pipeline (pipeline.jl):
   using PromptingTools
using JSON3
using Decimals

# API key configuration (or export in the terminal: OPENAI_API_KEY).
# ENV["OPENAI_API_KEY"] = "sua-chave-aqui"

# =====================================================================
# PASSO 1: Definir os Schemas de Extração (Tipagem Nativa Julia)
# =====================================================================

# Struct que o Agente Analista preencherá automaticamente via LLM Schema Extraction
Base.@kwdef struct VariavelCOBOL
    nome::String
    tipo_cobol::String
    tipo_julia_sugerido::String
    descricao::String
end

Base.@kwdef struct AnaliseCOBOL
    nome_programa::String
    objetivo::String
    variaveis::Vector{VariavelCOBOL}
    regras_de_negocio::Vector{String}
end

# =====================================================================
# PASSO 2: Código COBOL de Entrada
# =====================================================================

codigo_cobol = """
       IDENTIFICATION DIVISION.
       PROGRAM-ID. CALC-LIMITE.
       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01 WS-RENDA-MENSAL    PIC 9(6)V99 VALUE 005000.00.
       01 WS-SCORE-CREDITO   PIC 9(03)    VALUE 750.
       01 WS-LIMITE-FINAL    PIC 9(7)V99.

       PROCEDURE DIVISION.
           IF WS-SCORE-CREDITO > 700 THEN
               COMPUTE WS-LIMITE-FINAL = WS-RENDA-MENSAL * 2.5
           ELSE
               COMPUTE WS-LIMITE-FINAL = WS-RENDA-MENSAL * 1.0
           END-IF.
           DISPLAY "LIMITE: " WS-LIMITE-FINAL.
           STOP RUN.
"""

# =====================================================================
# PASSO 3: Agente Analista (Extração de Estrutura)
# =====================================================================

function agente_analista(codigo::String)
    println(" -> [Agente Analista] Mapeando estrutura do COBOL...")
    
    prompt = """
    Você é um especialista em Mainframe e COBOL.
    Análise o código abaixo e extraia o nome do programa, suas variáveis e as regras de negócio:
    
    $(codigo)
    """
    
    # aextract força o modelo a retornar diretamente a struct AnaliseCOBOL tipada
    msg = aextract(prompt; response_model=AnaliseCOBOL, model="gpt-4o-mini")
    return msg.content
end

# =====================================================================
# PASSO 4: Agente Tradutor (Geração de Código Julia)
# =====================================================================

function agente_tradutor(analise::AnaliseCOBOL)
    println(" -> [Agente Tradutor] Gerando código Julia idiomático e tipado...")
    
    prompt = """
    Você é um Engenheiro de Software Julia Sênior.
    Com base na seguinte análise de um programa COBOL, gere o código Julia equivalente.

    Regras obrigatórias:
    - Use o pacote `Decimals` para campos de valor monetário (evitando float).
    - Crie structs imutáveis para os dados.
    - Crie funções puras e bem tipadas.

    Análise COBOL:
    Programa: $(analise.nome_programa)
    Objetivo: $(analise.objetivo)
    Regras: $(join(analise.regras_de_negocio, ", "))
    """

    # aigenerate gera a resposta em texto/código estruturado
    msg = aigenerate(prompt; model="gpt-4o-mini")
    return msg.content
end

# =====================================================================
# PASSO 5: Execução do Pipeline Orquestrado
# =====================================================================

function executar_pipeline()
    println("=== INICIANDO PIPELINE DE AGENTES EM JULIA ===")
    
    # 1. Executa o Agente Analista
    analise = agente_analista(codigo_cobol)
    
    println("\n--- Mapeamento Concluído pelo Agente 1 ---")
    println("Programa: ", analise.nome_programa)
    println("Regras encontradas: ", length(analise.regras_de_negocio))
    
    # 2. Executa o Agente Tradutor
    codigo_julia_gerado = agente_tradutor(analise)
    
    println("\n================ CÓDIGO JULIA GERADO ================")
    println(codigo_julia_gerado)
end

executar_pipeline()


 Why use PromptingTools.jl instead of porting CrewAI?
Zero Overhead: PromptingTools.jl does not attempt to reimplement complex graph or agent list abstractions in an object-oriented framework. It treats agents as pure functions that transform data.


aextract with Julia Macros: The aextract function uses Julia’s metaprogramming system to inspect the fields of a struct and automatically generate the OpenAI/Anthropic Function Calling JSON schema.


Multiple Dispatch for Tools: You can extend the agent’s behavior simply by defining new function methods for new input types.
