import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def converter_para_dict(json_data):
    """ Converte JSON para dicionário antes de manipular. """
    if isinstance(json_data, str):
        return json.loads(json_data)  # Se vier como string, converte para dicionário
    return json_data  # Se já for dict, mantém

def formatar_json(obj):
    """ Converte o dicionário final para JSON formatado. """
    return json.dumps(obj, indent=4, ensure_ascii=False)

def remover_ufs(json_data):
    """ Remove a chave que contém as UFs e agrega os valores em uma lista única. """
    if isinstance(json_data, dict):
        return {key: ", ".join(value) if isinstance(value, list) else value for key, value in json_data.items() if "uf" not in key.lower()}
    return json_data

def get_city_data(cidade, uf):
    # Obtendo dados da tabela demarcation
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        return None

    demarcation_dict = converter_para_dict(demarcation_data.json_body if demarcation_data else {})
    filtrando_demarcation_dict = remover_ufs(demarcation_dict)  # Remove UFs e agrega dados

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
    filtrando_cluster_dict = remover_ufs(cluster_dict)  # Remove UFs e organiza os valores

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
