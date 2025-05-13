import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def get_city_data(cidade, uf):
    print(f"Processando cidade: {cidade}, UF: {uf}")

    # Obtendo dados da demarcação
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        return None

    # Obtendo dados do cluster
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

    # **Removendo a chave 'regions' do JSON antes de apresentar**
    if "regions" in cluster_json:
        del cluster_json["regions"]

    # Obtendo dados nacionais
    tipo_nacional = "dsp" if uf == "SP" else "fsp"
    nacional_data = TbMcastNacional.objects.filter(dsp_fsp=tipo_nacional).first()
    nacional_json = json.loads(nacional_data.json_body) if nacional_data else {}

    # Construindo o JSON final sem 'regions'
    city_data = {
        "city": cidade,
        "cluster": cluster_json,  # Agora sem 'regions'
        "demarcation": json.loads(demarcation_data.json_body),
        "federative Unit": demarcation_data.uf,
        "nacional": nacional_json,
        "nlc": demarcation_data.sigla
    }

    # Debug para verificar saída final
    print(f"JSON final filtrado: {city_data}")

    return city_data
