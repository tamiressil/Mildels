todas_as_regioes = TbMcastCluster.objects.all()
cluster_data = next((regiao for regiao in todas_as_regioes if uf.strip().lower() in {r.strip().lower() for r in regiao.region.split("_")}), None)
