for regiao in todas_as_regioes:
    regioes_list = [r.strip().lower() for r in regiao.region.split("_")]  # Remove espaços e coloca tudo em minúsculas
    uf_formatada = uf.strip().lower()  # Deixa a UF formatada corretamente
    print(f"Verificando se '{uf_formatada}' está dentro de {regioes_list}")  # 🛠 Debugging

    if uf_formatada in regioes_list:
        cluster_data = regiao
        break
