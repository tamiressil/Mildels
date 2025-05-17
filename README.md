import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def converter_para_dict(json_data):
    """ Garante que JSONs vindos como string sejam convertidos corretamente para dicionário. """
    if isinstance(json_data, str):
        return json.loads(json_data)  # Converte string JSON para dicionário
    return json_data  # Se já for dict, mantém

def formatar_json(obj):
    """ Formata JSON com indentação para melhor legibilidade. """
    return json.dumps(obj, indent=4, ensure_ascii=False)

def remover_ufs(json_data):
    """ Remove qualquer chave que contenha UFs dentro do JSON do cluster. """
    if isinstance(json_data, dict):
        return {key: value for key, value in json_data.items() if "uf" not in key.lower() and "region" not in key.lower()}
    return json_data

def remover_sigla_demarcation(json_data, sigla_para_remover):
    """ Remove a sigla dentro do JSON de demarcation sem apagar outros dados. """
    if isinstance(json_data, dict) and sigla_para_remover in json_data:
        del json_data[sigla_para_remover]
    return json_data

def get_city_data(cidade, uf):
    # Obtendo dados da tabela demarcation
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        return None

    demarcation_dict = converter_para_dict(demarcation_data.json_body if demarcation_data else {})
    sigla_para_remover = demarcation_data.sigla if demarcation_data else None
    filtrando_demarcation_dict = remover_sigla_demarcation(demarcation_dict, sigla_para_remover)

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
    filtrando_cluster_dict = remover_ufs(cluster_dict)  # Remove UFs corretamente

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
