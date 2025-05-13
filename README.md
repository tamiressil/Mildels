if cluster_data:
    cluster_json = json.loads(cluster_data.json_body)

    # 🚀 Remover qualquer referência às regiões dentro do JSON
    region_keys = ["pa_to_ma_ap", "sp_rj_mg_es", "pc_sc_rs", "al_ba_ce_pb_pe_pi_rn_se", "mt_ms_go_df", "am"]

    # 🔹 Criamos um novo JSON filtrado, excluindo as regiões
    cluster_filtrado = {key: value for key, value in cluster_json.items() if key not in region_keys}
else:
    cluster_filtrado = {}


    {% for item in items %}
    <div class="Quadrado_dados">
        <h1>{{ item.city }}</h1>
        {% for key, value in item.cluster.items %}
            <div class="dados_linha">
                <strong>{{ key }}:</strong> {{ value }}
            </div>
        {% endfor %}
    </div>
{% endfor %}
