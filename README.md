import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def get_city_data(cidade, uf):
    print(f"Processando cidade: {cidade}, UF: {uf}")

    # tb_mcast_demarcation
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        return None

    # tb_mcast_cluster
    todas_as_regioes = TbMcastCluster.objects.all()
    cluster_data = None
    for regiao in todas_as_regioes:
        regioes_list = {r.strip().lower() for r in regiao.region.split("_")}
        uf_formatada = uf.strip().lower()
        
        if uf_formatada in regioes_list:
            cluster_data = regiao
            break

    cluster_json = json.loads(cluster_data.json_body) if cluster_data else {}

    # **Filtrando o JSON para remover UFs**
    cluster_json_filtrado = {chave: valor for chave, valor in cluster_json.items() if not "_" in chave}

    # tb_mcast_nacional
    tipo_nacional = "dsp" if uf == "SP" else "fsp"
    nacional_data = TbMcastNacional.objects.filter(dsp_fsp=tipo_nacional).first()
    nacional_json = json.loads(nacional_data.json_body) if nacional_data else {}

    # Construindo city_data mantendo a estrutura original
    city_data = {
        "city": cidade,
        "cluster": cluster_json_filtrado,  # Agora sem UFs dentro do JSON do Cluster
        "demarcation": json.loads(demarcation_data.json_body),
        "federative Unit": demarcation_data.uf,
        "nacional": nacional_json,
        "nlc": demarcation_data.sigla
    }

    print(f"JSON final filtrado: {city_data}")
    return city_data 
