import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def get_city_data(cidade, uf):
    # Obtendo dados de demarcation
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        return None

    demarcation_json = json.loads(json.dumps(demarcation_data.json_body)) if demarcation_data else {}
    filtrando_demarcation_json = {key: value for key, value in demarcation_json.items() if key != 'sigla'}

    # Obtendo dados do cluster
    todas_as_regioes = TbMcastCluster.objects.all()
    cluster_data = next((regiao for regiao in todas_as_regioes if uf.strip().lower() in {r.strip().lower() for r in regiao.region.split("_")}), None)

    cluster_json = json.loads(json.dumps(cluster_data.json_body)) if cluster_data else {}
    filtrando_cluster_json = {key: value for key, value in cluster_json.items() if key != 'region'}

    # Obtendo dados da tabela nacional
    tipo_nacional = "dsp" if uf == "SP" else "fsp"
    nacional_data = TbMcastNacional.objects.filter(dsp_fsp=tipo_nacional).first()
    nacional_json = json.loads(json.dumps(nacional_data.json_body)) if nacional_data else {}

    filtrando_nacional_json = {key: value for key, value in nacional_json.items() if key == "channels"}

    # Formatando a saída JSON com melhor estruturação
    city_data = json.dumps({
        "city": cidade,
        "cluster": [[subvalue] for subvalue in filtrando_cluster_json.values()],  
        "demarcation": [[subvalue] for subvalue in filtrando_demarcation_json.values()],  
        "federative_Unit": demarcation_data.uf.lower() if demarcation_data else None,
        "nacional": [[subvalue] for subvalue in filtrando_nacional_json.values()],  
        "nlc": demarcation_data.sigla.lower() if demarcation_data else None,
    }, indent=4)  # Adicionando espaçamento para melhorar a visualização

    return city_data

