
    import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def get_city_data(cidade, uf):
    print(f"Processando cidade: {cidade}, UF: {uf}")

    # Obtendo dados do Cluster
    todas_as_regioes = TbMcastCluster.objects.all()
    cluster_data = None
    for regiao in todas_as_regioes:
        regioes_list = {r.strip().lower() for r in regiao.region.split("_")}
        uf_formatada = uf.strip().lower()
        
        if uf_formatada in regioes_list:
            cluster_data = regiao
            break

    cluster_json = json.loads(cluster_data.json_body) if cluster_data else {}

    # **Filtrando o JSON para remover chaves com UFs**
    cluster_json_sem_ufs = {chave: valor for chave, valor in cluster_json.items() if not "_" in chave}

    # Debug para conferir antes e depois
    print(f"JSON antes da filtragem: {cluster_json}")
    print(f"JSON depois da filtragem: {cluster_json_sem_ufs}")

    # Retornando JSON filtrado
    return {
        "city": cidade,
        "cluster": cluster_json_sem_ufs,
        "demarcation": json.loads(TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first().json_body),
        "federative Unit": uf,
    }
