import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def converter_para_dict(json_data):
    """ Converte um JSON para um dicionário Python antes de manipular. """
    if isinstance(json_data, str):  # Se vier como string, convertemos
        return json.loads(json_data)
    return json_data  # Se já for dict, mantemos

def formatar_json(obj):
    """ Converte o dicionário final para JSON formatado. """
    return json.dumps(obj, indent=4, ensure_ascii=False)

def get_city_data(cidade, uf):
    # Obtendo dados da tabela demarcation
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        return None

    demarcation_dict = converter_para_dict(demarcation_data.json_body if demarcation_data else {})
    filtrando_demarcation_dict = {key: value for key, value in demarcation_dict.items() if key != 'sigla'}

    # Obtendo dados do cluster
    todas_as_regioes = TbMcastCluster.objects.all()
    cluster_data = None
    for regiao in todas_as_regioes:
        regioes_list = {r.strip().lower() for r in regiao.region.split("_")}
        uf_formatada = uf.strip().lower()

        if uf_formatada in regioes_list:
            cluster_data = regiao
            break

    cluster_dict = converter_para_dict(cluster_data.json_body if cluster_data else {})
    filtrando_cluster_dict = {key: value for key, value in cluster_dict.items() if key != 'region'}

    # Obtendo dados nacionais
    tipo_nacional = "dsp" if uf == "SP" else "fsp"
    nacional_data = TbMcastNacional.objects.filter(dsp_fsp=tipo_nacional).first()
    nacional_dict = converter_para_dict(nacional_data.json_body if nacional_data else {})
    filtrando_nacional_dict = {key: value for key, value in nacional_dict.items() if key == "channels"}

    # Criando dicionário final
    city_data = {
        "city": cidade,
        "cluster": filtrando_cluster_dict,
        "demarcation": filtrando_demarcation_dict,
        "federative_Unit": demarcation_data.uf.lower() if demarcation_data else None,
        "nacional": filtrando_nacional_dict,
        "nlc": demarcation_data.sigla.lower() if demarcation_data else None,
    }

    return formatar_json(city_data)



