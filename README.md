import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

# Lista de regiões que queremos remover do JSON
regioes_para_remover = ["SP", "RJ", "MG"]  # Adicione as que desejar

def get_city_data(cidade, uf):
    print(f"Processando cidade: {cidade}, UF: {uf}")

    # Obtendo dados da demarcação
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        print("Nenhuma demarcação encontrada.")
        return None

    # Obtendo dados do cluster
    todas_as_regioes = TbMcastCluster.objects.all()
    cluster_data = None
    for regiao in todas_as_regioes:
        regioes_list = [r.strip().lower() for r in regiao.region.split("_")]
        uf_formatada = uf.strip().lower()
        if uf_formatada in regioes_list:
            cluster_data = regiao
            break

    cluster_json = json.loads(cluster_data.json_body) if cluster_data else {}

    # Removendo a chave 'region', se existir
    if 'region' in cluster_json:
        del cluster_json['region']

    # **Filtrando as regiões indesejadas**
    if 'regios' in cluster_json:
        cluster_json["regios"] = [r for r in cluster_json["regios"] if r not in regioes_para_remover]

    # Obtendo dados nacionais
    tipo_nacional = "dsp" if uf == "SP" else "fsp"
    nacional_data = TbMcastNacional.objects.filter(dsp_fsp=tipo_nacional).first()
    nacional_json = json.loads(nacional_data.json_body) if nacional_data else {}

    # Construindo o JSON final
    city_data = {
        "city": cidade,
        "cluster": cluster_json,
        "demarcation": json.loads(demarcation_data.json_body),
        "federative Unit": demarcation_data.uf,
        "nacional": nacional_json,
        "nlc": demarcation_data.sigla
    }

    print(f"Dados finais filtrados para cidade {cidade}: {city_data}")
    return city_data


