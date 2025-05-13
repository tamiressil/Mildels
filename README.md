if cluster_data:
    cluster_json = json.loads(cluster_data.json_body)

    # 🔎 Teste para ver o JSON original antes da filtragem
    print(f"JSON original do cluster: {cluster_json}")

    # 🚀 Filtrar a UF dentro da lista de regiões e remover referências extras
    cluster_filtrado = next((data for key, data in cluster_json.items() if uf in key), {})

    print(f"Cluster filtrado para UF {uf}: {cluster_filtrado}")
else:
    cluster_filtrado = {}
