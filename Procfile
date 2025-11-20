import unicodedata
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional

app = FastAPI(title="API de Triagem Odontológica")
app = FastAPI(title="API-Ruth")

# --- BANCO DE DADOS DOS PROFISSIONAIS ---
PROFESSIONALS = {
    "Dayara": {"id": 4773939817545728, "name": "Dayara Boscolo", "keywords": ["dayara"]},
    "Ramon": {"id": 5108599479861248, "name": "Ramon Uchoa dos Anjos", "keywords": ["ramon", "uchoa"]},
    "Vinicius": {"id": 5478954060808192, "name": "Vinicius Targino Gomes de Almeida", "keywords": ["vinicius", "targino"]},
    "Gabriela": {"id": 5859536659349504, "name": "Gabriela Formiga da Silva", "keywords": ["gabriela", "formiga", "gabi"]},
    "Ruth": {"id": 5897012130873344, "name": "Maria Ruth Costa Rodrigues", "keywords": ["ruth", "maria ruth"]},
    "Katianne": {"id": 6068925041999872, "name": "Katianne Gomes Dias Bezerra", "keywords": ["katianne", "katiane", "kati"]},
    "Mateus": {"id": 6462444026265600, "name": "Mateus Correia Vidal Ataide", "keywords": ["mateus", "matheus", "ataide"]},
    "Camylla": {"id": 6567447868735488, "name": "Camylla Farias Brandão", "keywords": ["camylla", "camila", "faria"]}
}

# --- MODELO DE ENTRADA ---
class ServiceRequest(BaseModel):
    service_text: str

# --- FUNÇÃO AUXILIAR PARA REMOVER ACENTOS ---
def normalize_text(text: str) -> str:
    # Transforma 'Extração' em 'extracao' e 'Siso' em 'siso'
    return ''.join(c for c in unicodedata.normalize('NFD', text)
                   if unicodedata.category(c) != 'Mn').lower()

# --- LÓGICA DE ROTEAMENTO ---
def find_professional(text: str):
    clean_text = normalize_text(text)
    
    # 1. VERIFICAÇÃO DE PREFERÊNCIA (Se o cliente citou o nome do dentista)
    for key, data in PROFESSIONALS.items():
        for keyword in data["keywords"]:
            if keyword in clean_text:
                return data

    # 2. ROTEAMENTO POR SERVIÇO (Palavras-chave)
    
    # Dra Camylla - Canal
    if "canal" in clean_text or "endodontia" in clean_text:
        return PROFESSIONALS["Camylla"]

    # Dra Katianne - Aparelho ou Botox
    if any(word in clean_text for word in ["aparelho", "orto", "botox", "harmonizacao"]):
        return PROFESSIONALS["Katianne"]

    # Dr Vinicius - Facetas ou Estética
    if any(word in clean_text for word in ["faceta", "lente", "estetica"]):
        return PROFESSIONALS["Vinicius"]
        
    # Dr Mateus - Extração (Siso ou Simples)
    # Prioridade para Mateus em extrações, a menos que especificado outro
    if "extracao" in clean_text or "siso" in clean_text or "arrancar" in clean_text:
        return PROFESSIONALS["Mateus"]

    # Dr Ramon - Prótese, Coroas, Gengivoplastia (Implantes também, mas compartilhado)
    if any(word in clean_text for word in ["protese", "coroa", "gengiva", "implante", "protocolo"]):
        # Nota: Dayara também faz implante, mas Ramon é o principal na lista para "Protese/Coroas/Implantes"
        return PROFESSIONALS["Ramon"]

    # Dra Gabriela - Urgência, Infantil
    if any(word in clean_text for word in ["urgencia", "dor", "infantil", "crianca", "kids", "pediatria"]):
        return PROFESSIONALS["Gabriela"]

    # --- ÁREA DE CONFLITOS (Clareamento, Limpeza, Restauração) ---
    # Dra Ruth vs Dra Gabriela
    
    common_procedures = ["clareamento", "limpeza", "profilaxia", "restauracao", "obtura", "dentistica"]
    
    if any(word in clean_text for word in common_procedures):
        # A regra diz: "Clareamento (paciente só quiser com a Dra Ruth)"
        # Isso implica que o padrão é Gabriela, a menos que Ruth seja citada (já checado no passo 1)
        return PROFESSIONALS["Gabriela"]

    # Se não entendeu nada, retorna None ou um padrão (Administrativo)
    return None

# --- ENDPOINT DA API ---
@app.post("/match-professional")
async def match_professional(request: ServiceRequest):
    professional = find_professional(request.service_text)
    
    if professional:
        return {
            "found": True,
            "professional_id": professional["id"],
            "professional_name": professional["name"],
            "original_request": request.service_text
        }
    else:
        # Caso não encontre, você pode retornar um erro ou um ID de secretária
        return {
            "found": False,
            "message": "Não foi possível identificar o serviço ou profissional automaticamente.",
            "original_request": request.service_text
        }

# Para rodar localmente: uvicorn main:app --reload
