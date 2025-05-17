import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def get_city_data(cidade, uf):
    # Dados da tabela demarcation
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        return None

    demarcation_json = demarcation_data.json_body if demarcation_data else {}  
    filtrando_demarcation_json = {key: value for key, value in demarcation_json.items() if key != 'sigla'}

    # Dados da tabela do cluster
    todas_as_regioes = TbMcastCluster.objects.all()
    cluster_data = next((regiao for regiao in todas_as_regioes if uf.strip().lower() in {r.strip().lower() for r in regiao.region.split("_")} if regiao.region else set()), None)

    cluster_json = cluster_data.json_body if cluster_data else {}  
    filtrando_cluster_json = {key: value for key, value in cluster_json.items() if key != 'region'}

    # Dados da tabela nacional
    tipo_nacional = "dsp" if uf == "SP" else "fsp"
    nacional_data = TbMcastNacional.objects.filter(dsp_fsp=tipo_nacional).first()
    nacional_json = nacional_data.json_body if nacional_data else {}

    filtrando_nacional_json = {key: value for key, value in nacional_json.items() if key == "channels"}

    # Ajustando formatação do JSON para melhor visualização, deixando os blocos organizados verticalmente
    city_data = {
        "city": cidade,
        "cluster": filtrando_cluster_json,  
        "demarcation": filtrando_demarcation_json,  
        "federative_Unit": demarcation_data.uf.lower() if demarcation_data else None,
        "nacional": filtrando_nacional_json,  
        "nlc": demarcation_data.sigla.lower() if demarcation_data else None,
    }

    return json.dumps(city_data, indent=4, separators=(", ", ": "), ensure_ascii=False)
