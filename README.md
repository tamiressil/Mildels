if cluster_data:
    cluster_json = json.loads(cluster_data.json_body)

    # 🚀 Se a UF da cidade estiver em alguma das regiões, remova os dados da região
    if any(uf in key for key in cluster_json.keys()):
        for key in list(cluster_json.keys()):
            if key.startswith("pa_to_ma_ap"):  # 🔹 Aqui, removemos a referência indesejada
                del cluster_json[key]

    # 🔍 Agora garantimos que os dados não tenham a região, só as informações relevantes
    dados_filtrados = [valor for valor in cluster_json.values()]

    print(f"Dados filtrados do cluster para UF {uf}: {dados_filtrados}")
else:
    dados_filtrados = []




    {% extends 'base.html' %}
{% load static %}

{% block title %}Detalhes da Cidade{% endblock %}

{% block content %}

<link rel="stylesheet" href="{% static 'simple_api/js/detalhes_cidade.css' %}">

{% for item in items %}
    <br>
    <h1>{{ item.city }}</h1>
    <br>
    <div class="Quadrado_dados">
        <br>
        <code>"city": {{ item.city }}</code><br>
        
        <!-- 🚀 Aqui, garantimos que o cluster aparece formatado corretamente -->
        {% for value in item.cluster_filtrado %}
            <code>{{ value }}</code><br>
        {% endfor %}

        <code>"demarcation": {{ item.demarcation }}</code><br>
        <code>"federative Unit": {{ item.federative_Unit }}</code><br>
        <code>"nacional": {{ item.nacional }}</code><br>
        <code>"nlc": {{ item.nlc }}</code>
    </div>
{% endfor %}

{% endblock %}
