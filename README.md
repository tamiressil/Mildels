body {
    background-color: black;
    color: white;
    font-family: Arial, sans-serif;
}

.titulo {
    font-size: 24px;
    text-align: center;
    color: #00aaff; /* Azul claro */
    padding: 10px;
}

.Quadrado_dados {
    background-color: #222;
    padding: 20px;
    border-radius: 5px;
    width: 90%;
    max-width: 600px;
    margin: auto;
    overflow-x: auto;
}

pre {
    white-space: pre-wrap;
    color: #00aaff;
    font-family: monospace;
    padding: 10px;
}




{% extends 'base.html' %}
{% load static %}

{% block title %}Detalhes da Cidade{% endblock %}

{% block content %}
<link rel="stylesheet" href="{% static 'static_files/js/css/detalhes_cidade.css' %}">

{% for item in items %}
    <br>
    <h1 class="titulo">{{ item.city }}</h1>

    <div class="Quadrado_dados">
        <pre><code>{{ item | safe }}</code></pre>
    </div>
{% endfor %}

{% endblock %}



import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def formatar_json(obj):
    """ Garante que o JSON seja formatado corretamente com espaçamento e indentação. """
    return json.dumps(obj, indent=4, ensure_ascii=False)

def get_city_data(cidade, uf):
    # Obtendo dados de demarcation
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        return None

    demarcation_json = demarcation_data.json_body if demarcation_data else {}
    filtrando_demarcation_json = {key: value for key, value in demarcation_json.items() if key != 'sigla'}

    # Obtendo dados do cluster
    todas_as_regioes = TbMcastCluster.objects.all()
    cluster_data = None
    for regiao in todas_as_regioes:
        regioes_list = {r.strip().lower() for r in regiao.region.split("_")}
        uf_formatada = uf.strip().lower()

        if uf_formatada in regioes_list:
            cluster_data = regiao
            break

    cluster_json = cluster_data.json_body if cluster_data else {}
    filtrando_cluster_json = {key: value for key, value in cluster_json.items() if key != 'region'}

    # Obtendo dados nacionais
    tipo_nacional = "dsp" if uf == "SP" else "fsp"
    nacional_data = TbMcastNacional.objects.filter(dsp_fsp=tipo_nacional).first()
    nacional_json = nacional_data.json_body if nacional_data else {}
    filtrando_nacional_json = {key: value for key, value in nacional_json.items() if key == "channels"}

    # Dados formatados
    city_data = {
        "city": cidade,
        "cluster": filtrando_cluster_json,
        "demarcation": filtrando_demarcation_json,
        "federative_Unit": demarcation_data.uf.lower() if demarcation_data else None,
        "nacional": filtrando_nacional_json,
        "nlc": demarcation_data.sigla.lower() if demarcation_data else None,
    }

    return formatar_json(city_data)import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def formatar_json(obj):
    """ Garante que o JSON seja formatado corretamente com espaçamento e indentação. """
    return json.dumps(obj, indent=4, ensure_ascii=False)

def get_city_data(cidade, uf):
    # Obtendo dados de demarcation
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        return None

    demarcation_json = demarcation_data.json_body if demarcation_data else {}
    filtrando_demarcation_json = {key: value for key, value in demarcation_json.items() if key != 'sigla'}

    # Obtendo dados do cluster
    todas_as_regioes = TbMcastCluster.objects.all()
    cluster_data = None
    for regiao in todas_as_regioes:
        regioes_list = {r.strip().lower() for r in regiao.region.split("_")}
        uf_formatada = uf.strip().lower()

        if uf_formatada in regioes_list:
            cluster_data = regiao
            break

    cluster_json = cluster_data.json_body if cluster_data else {}
    filtrando_cluster_json = {key: value for key, value in cluster_json.items() if key != 'region'}

    # Obtendo dados nacionais
    tipo_nacional = "dsp" if uf == "SP" else "fsp"
    nacional_data = TbMcastNacional.objects.filter(dsp_fsp=tipo_nacional).first()
    nacional_json = nacional_data.json_body if nacional_data else {}
    filtrando_nacional_json = {key: value for key, value in nacional_json.items() if key == "channels"}

    # Dados formatados
    city_data = {
        "city": cidade,
        "cluster": filtrando_cluster_json,
        "demarcation": filtrando_demarcation_json,
        "federative_Unit": demarcation_data.uf.lower() if demarcation_data else None,
        "nacional": filtrando_nacional_json,
        "nlc": demarcation_data.sigla.lower() if demarcation_data else None,
    }

    return formatar_json(city_data)
