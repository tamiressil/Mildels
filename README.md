if cluster_data:
    print(f"Cluster encontrado para {cidade}, {uf}: {cluster_data.json_body}")
else:
    print(f"NENHUM CLUSTER encontrado para {cidade}, {uf}!")

    for regiao in todas_as_regioes:
    regioes_list = regiao.region.split("_")  # Divide "SP_RJ_MG_ES" em ['SP', 'RJ', 'MG', 'ES']
    print(f"Região analisada: {regioes_list}")  # 🔍 Veja quais UFs aparecem
    if uf.strip() in [r.strip() for r in regioes_list]:  # Remove espaços e verifica corretamente
        cluster_data = regiao
        break
