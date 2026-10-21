# pecu-ria-app
Aplicativo para pecuáriastas tirarem suas duvidas apenas sobre isso.
import json
import re
from pathlib import Path
from datetime import datetime

ARQUIVO_MEMORIA = Path("memoria_pecuaria.json")

# BASE DE CONHECIMENTO - pode aumentar quanto quiser
BASE_CONHECIMENTO = {
    "manejo": """
🐂 MANEJO:
- Curral: Sempre com sombra e água limpa.
- Lote por categoria: bezerro, recria, engorda, matriz.
- Rotação de pasto: piquete descansa 28-35 dias.
- Cocho: 30cm linear por cabeça para suplemento.
""",
    "nutricao": """
🌾 NUTRIÇÃO / TRATO:
- RIP MAX 23: proteico para época seca.
- DIETA TOTAL: mistura completa (volumoso + concentrado).
- Sal mineral sempre à vontade.ss
- Água: boi bebe 40 a 80 litros/dia. Se faltar água, cai o ganho.
- Regra de trato: 1,5% a 2,5% do peso vivo em matéria seca.
""",
    "sanidade": """
💉 SANIDADE:
- Vermifugação: Bullmax, Ivomec, etc - seguir bula.
- Vacinas obrigatórias: Aftosa, Raiva, Clostridiose.
- Sempre anote: data, brinco, remédio, dose (igual você faz).
- Bezerro: curar umbigo + colostro nas primeiras 6h.
""",
    "chuva": """
🌧️ PLUVIÔMETRO:
- Cada palitinho pequeno = 2mm
- Cada traço grande (10,20,30) = 10mm
- Acima de 20mm já ajuda o broto do capim.
""",
    "nelore": """
🐃 NELORE:
- Rústico, tolera calor.
- Ganho médio em confinamento: 1,4 a 1,8 kg/dia.
- Cruzado: ganha mais em acabamento, mas exige mais nutrição.
""",
    "confinamento": """
🏠 CONFINAMENTO:
- Adaptação 14-21 dias.
- Dieta: começar com mais volumoso, ir aumentando concentrado.
- Leitura de cocho todo dia cedo.
"""
}

PALAVRAS_CHAVE = {
    "manejo": ["manejo", "piquete", "curral", "pasto", "lote", "cabeceira", "fundo"],
    "nutricao": ["trato", "dieta", "rip", "ração", "racao", "sal", "saco", "kg", "cocho", "nutrição", "nutricao", "comida"],
    "sanidade": ["remédio", "remedio", "vacina", "vermifugo", "bullmax", "cordenco", "doença", "doenca", "carrapato", "berne"],
    "chuva": ["chuva", "pluviometro", "mm", "choveu"],
    "nelore": ["nelore", "cruzamento", "cruzado", "boi", "gado", "bezerro", "angus"],
    "confinamento": ["confinamento", "engorda", "ganho", "peso", "nutrição"]
}

PALAVRAS_PROIBIDAS = [] # aqui você bloqueia o que NÃO é pecuária

def carregar_memoria():
    if ARQUIVO_MEMORIA.exists():
        try:
            return json.loads(ARQUIVO_MEMORIA.read_text(encoding="utf-8"))
        except: return []
    return []

def salvar_memoria(hist):
    ARQUIVO_MEMORIA.write_text(json.dumps(hist, ensure_ascii=False, indent=2), encoding="utf-8")

def eh_sobre_pecuaria(texto):
    t = texto.lower()
    # Se perguntar sobre política, futebol, etc, bloqueia
    bloqueados = ["política", "futebol", "bbb", "game", "filme", "bitcoin", "lula", "bolsonaro"]
    if any(p in t for p in bloqueados):
        return False
    # Se tiver qualquer palavra da pecuária, libera
    for lista in PALAVRAS_CHAVE.values():
        if any(p in t for p in lista):
            return True
    # Perguntas curtas tipo /ajuda libera
    if t.startswith("/"): return True
    # Se não achou palavra, pede pra especificar mas ainda considera pecuária
    return True

def responder(pergunta):
    p = pergunta.lower()
    # Procura tema
    for tema, palavras in PALAVRAS_CHAVE.items():
        if any(re.search(r"\b" + re.escape(pal) + r"\b", p) for pal in palavras):
            return BASE_CONHECIMENTO[tema]
    
    return """
Não entendi bem, mas tô aqui só pra pecuária.

Pode perguntar sobre:
- trato / dieta / RIP MAX
- manejo de lote (cabeceira/fundo)
- chuva / pluviômetro
- remédio / vacina / vermífugo
- nelore / cruzamento / confinamento

Ex: "quantos kg de trato pra 50 cabeças de nelore?"
"""
    historico = carregar_memoria()
    print("="*60)
    print("  MIRANHA PECUÁRIA - CHAT GRATUITO E INFINITO")
    print("="*60)
    print("Só respondo sobre pecuária. Digite /sair pra fechar.")
    print(f"Memória: {len(historico)} msgs salvas.\n")

    while True:
        try:
            pergunta = input("Você: ").strip()
        except KeyboardInterrupt:
            break
        if not pergunta: continue
        if pergunta.lower() == "/sair": break
        if pergunta.lower() == "/limpar":
            historico = []
            if ARQUIVO_MEMORIA.exists(): ARQUIVO_MEMORIA.unlink()
            print("Memória limpa!\n")
            continue
        if pergunta.lower() == "/ajuda":
            print(responder("/ajuda"))
            continue

        if not eh_sobre_pecuaria(pergunta):
            print("\nBOT: Só posso falar sobre PECUÁRIA. Me pergunta sobre trato, gado, pasto, chuva, remédio.\n")
            continue

        resp = responder(pergunta)
        print(f"\nBOT PECUÁRIA: {resp}\n")

        historico.append({"data": datetime.now().isoformat(), "pergunta": pergunta, "resposta": resp})
        salvar_memoria(historico[-100:]) # guarda últimas 100

    print("\nAté amanhã no curral!")

import json, re
from pathlib import Path
from datetime import datetime

ARQUIVO = Path("memoria_pecuaria.json")

# ================= MODO 1: PECUÁRIA (para o cliente) =================
BASE_PECUARIA = {
    "manejo": "🐂 Lote por categoria, piquete descansa 28-35 dias, 30cm de cocho por cabeça.",
    "nutricao": "🌾 Trato 1,5% a 2,5% PV. RIP MAX 23 na seca, DIETA TOTAL no confinamento. Água 40-80L/dia.",
    "sanidade": "💉 Anote sempre brinco, dose, data. Vacinas: Aftosa, Raiva, Clostridiose.",
    "chuva": "🌧️ Cada palitinho = 2mm, traço grande = 10mm.",
    "nelore": "🐃 Nelore rústico 1,4-1,8kg/dia. Cruzado ganha mais mas come mais.",
}

# ================= MODO 2: VENDAS DE MENTORIA (para VOCÊ) =================
# Aqui é o pulo do gato: você cola a msg do cliente e ele te dá a resposta

ROTEIRO_VENDAS = {
    "ligacao": {
        "gatilhos": ["ligação", "trabalha", "faz o que", "lida"],
        "resposta": "Boa! E você tem alguma ligação com pecuária hoje? Já mexe com gado ou tá começando do zero?"
    },
    "tipo_criacao": {
        "gatilhos": ["tenho", "gado", "animais", "fazenda", "hectares", "alqueire", "sítio"],
        "resposta": "Bacana, {nome}. E essa criação que você tem hoje é mais de cria, recria ou engorda? E quantas cabeças você tá rodando hoje?"
    },
    "dor_tecnica": {
        "gatilhos": ["lotação", "quantos", "caberia", "sustentar", "irrigação", "custo", "produtividade", "paga", "nutrição", "dieta", "suplementação", "formular", "tratos", "não ganho", "não engorda"],
        "resposta": """Entendi. Sua dúvida é bem comum: quantos animais essa área conseguiria sustentar, quanto custaria implantar a irrigação ou se o ganho de produtividade pagaria esse investimento?

O que mais te trava hoje: é a parte de NUTRIÇÃO (saber formular suplementação/dieta) ou de GESTÃO (saber se o investimento se paga)?"""
    },
    "objeção_empresa": {
        "gatilhos": ["empresa", "contratar", "consultoria", "técnico", "caro", "pensar", "definido"],
        "resposta": """Faz sentido. Antes de contratar uma empresa, o que você gostaria de ter definido por conta própria?

Muita gente que chega na minha mentoria queria justamente isso: chegar na empresa já sabendo calcular lotação, formular dieta e fazer a conta do ROI da irrigação, pra não ficar refém.

Quer que eu te mostre como a gente faz isso em 3 passos na mentoria?"""
    },
    "fechamento": {
        "gatilhos": ["quanto custa", "valor", "preço", "mentoria", "como funciona", "interesse"],
        "resposta": """A mentoria é direta ao ponto: te ensino a calcular lotação da sua área, formular sua dieta total/RIP e fazer a conta se a irrigação se paga - tudo com planilha que já uso na fazenda.

Não é curso gravado, é acompanhamento. Hoje tenho 2 vagas.

Quer que eu te mande um áudio de 2 min explicando como seria no SEU caso ou já quer ver a planilha de lotação?"""
    }
}

def detectar_intencao_venda(msg_cliente):
    msg = msg_cliente.lower()
    for tipo, dados in ROTEIRO_VENDAS.items():
        if any(g in msg for g in dados["gatilhos"]):
            return tipo
    return "ligacao" # padrão é quebrar gelo

def gerar_resposta_venda(msg_cliente):
    # tenta pegar nome
    nome = "Cleber"
    m = re.search(r"(?:sou|meu nome é|aqui é o)\s+(\w+)", msg_cliente, re.I)
    if m: nome = m.group(1).capitalize()
    
    tipo = detectar_intencao_venda(msg_cliente)
    template = ROTEIRO_VENDAS[tipo]["resposta"]
    return template.format(nome=nome), tipo

def salvar(msg):
    hist = []
    if ARQUIVO.exists():
        try: hist = json.loads(ARQUIVO.read_text(encoding="utf-8"))
        except: pass
    hist.append(msg)
    ARQUIVO.write_text(json.dumps(hist[-200:], ensure_ascii=False, indent=2), encoding="utf-8")

# ================= LOOP PRINCIPAL =================
def main():
    print("="*65)
    print("  🐂 MIRANHA - PECUÁRIA + VENDAS DE MENTORIA")
    print("="*65)
    print("COMO USAR:")
    print("1. Se for DÚVIDA DE PECUÁRIA, digite normal")
    print("2. Se for MENSAGEM COPIADA DO CLIENTE, digite: cliente: + cola a msg")
    print("   Ex: cliente: tenho 30 hectares e queria saber de irrigação")
    print("3. /sair pra fechar\n")

    while True:
        entrada = input("Você: ").strip()
        if not entrada: continue
        if entrada.lower() == "/sair": break

        # MODO VENDAS
        if entrada.lower().startswith("cliente:"):
            msg_cliente = entrada[8:].strip()
            resposta_pronta, etapa = gerar_resposta_venda(msg_cliente)
            print(f"\n[CLIENTE DISSE] -> {msg_cliente}")
            print(f"[ETAPA DETECTADA] -> {etapa.upper()}")
            print(f"\n✅ RESPOSTA PRONTA PRA COPIAR E MANDAR:\n")
            print(f"{resposta_pronta}\n")
            print("-"*65 + "\n")
            salvar({"tipo":"venda","cliente":msg_cliente,"resposta":resposta_pronta,"data":datetime.now().isoformat()})
            continue

        # MODO PECUÁRIA
        p = entrada.lower()
        achou = False
        for k,v in BASE_PECUARIA.items():
            if k in p or any(x in p for x in ["trato","rip","nelore","chuva","remedio","pasto"]):
                print(f"\nBOT PECUÁRIA: {v}\n")
                achou=True
                break
        if not achou:
            print("\nBOT PECUÁRIA: Me pergunta sobre trato, dieta, RIP MAX, chuva, remédio ou nelore/cruzado.\n")
            print("DICA DE VENDA: Se colar msg do cliente use 'cliente: mensagem dele'\n")
    
if __name__ == "__main__":
    main()
