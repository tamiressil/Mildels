import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def get_city_data(cidade, uf):
    print(f"Processando cidade: {cidade}, UF: {uf}")

    # Obtendo os dados da demarcação
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        return None

    # Obtendo os dados do cluster
    todas_as_regioes = TbMcastCluster.objects.all()
    cluster_data = None
    for regiao in todas_as_regioes:
        regioes_list = {r.strip().lower() for r in regiao.region.split("_")}
        uf_formatada = uf.strip().lower()
        
        if uf_formatada in regioes_list:
            cluster_data = regiao
            break

    # Se cluster_data existir, carregar JSON
    cluster_json = json.loads(cluster_data.json_body) if cluster_data else {}

    # **Garantindo que apenas as UFs dentro do JSON do Cluster sejam removidas**
    if cluster_json:
        cluster_json_filtrado = {chave: valor for chave, valor in cluster_json.items() if "_" not in chave}
    else:
        cluster_json_filtrado = {}

    # Obtendo os dados nacionais
    tipo_nacional = "dsp" if uf == "SP" else "fsp"
    nacional_data = TbMcastNacional.objects.filter(dsp_fsp=tipo_nacional).first()
    nacional_json = json.loads(nacional_data.json_body) if nacional_data else {}

    # Construindo o JSON final mantendo **federative Unit** e **cluster**
    city_data = {
        "city": cidade,
        "cluster": cluster_json_filtrado if cluster_json_filtrado else {},  # Garante que não fique vazio
        "demarcation": json.loads(demarcation_data.json_body) if demarcation_data else {},
        "federative Unit": demarcation_data.uf if demarcation_data else "",
        "nacional": nacional_json,
        "nlc": demarcation_data.sigla if demarcation_data else ""
    }

    # Debug para verificar saída final
    print(f"JSON final filtrado: {city_data}")

    return city_data
