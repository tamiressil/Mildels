
import json
import re
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def adicionar_espacamento(texto):
    # Adicionar espaçamento ao redor dos colchetes, aspas simples e vírgulas
    texto = re.sub(r"($$|^|\')", r" \1 ", texto)
    texto = re.sub(r",", r" , ", texto)
    return texto

def processar_json(obj):
    if isinstance(obj, dict):
        return {k: processar_json(v) for k, v in obj.items()}
    elif isinstance(obj, list):
        return [processar_json(item) for item in obj]
    elif isinstance(obj, str):
        return adicionar_espacamento(obj)
    else:
        return obj

def get_city_data(cidade, uf):
    # Dados da tabela demarcation
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        return None

    demarcation_json = json.loads(json.dumps(demarcation_data.json_body)) if demarcation_data else {}
    filtrando_demarcation_json = {key: value for key, value in demarcation_json.items() if key != 'sigla'}

    # Dados da tabela do cluster
    todas_as_regioes = TbMcastCluster.objects.all()
    cluster_data = None
    for regiao in todas_as_regioes:
        regioes_list = {r.strip().lower() for r in regiao.region.split("_")}
        uf_formatada = uf.strip().lower()

        if uf_formatada in regioes_list:
            cluster_data = regiao
            break

    cluster_json = json.loads(json.dumps(cluster_data.json_body)) if cluster_data else {}
    filtrando_cluster_json = {key: value for key, value in cluster_json.items() if key != 'region'}

    # Tabela nacional
    tipo_nacional = "dsp" if uf == "SP" else "fsp"
    nacional_data = TbMcastNacional.objects.filter(dsp_fsp=tipo_nacional).first()
    nacional_json = json.loads(json.dumps(nacional_data.json_body)) if nacional_data else {}
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

    # Processar o JSON para adicionar espaçamento
    city_data_processada = processar_json(city_data)

    return city_data_processada 

    {% extends 'base.html' %}
{% load static %}

{% block title %}Detalhes da Cidade{% endblock %}

{% block content %}
<link rel="stylesheet" href="{% static 'static_files/js/css/detalhes_cidade.css' %}">

{% for item in items %}
    <br>
    <br>
    <h1>{{ item.city }}</h1>

    <div class="Quadrado_dados">
        <br>
        <br>
        <code class="city">"city":<span> {{ item.city }}</span></code>
        <br><br>

        <code>"cluster":</code><br><br>
        {% for value in item.cluster.values %}
            {% for subvalue in value %}
                <code class="data-item"><span>{{ subvalue }}</span></code><br>
            {% endfor %}
        {% endfor %}
        <br>

        <code class="demarcation">"demarcation":</code>
        <br><br>
        {% for value in item.demarcation.values %}
            {% for subvalue in value %}
                <code class="data-item"><span>{{ subvalue }}</span></code><br>
            {% endfor %}
        {% endfor %}
        <br>

        <code id="federative_unit">"federative Unit": <span>"{{ item.federative_Unit }}"</span> </code>
        <br><br>

        <code class="nacional">"nacional":</code>
        <br><br>
        {% for value in item.nacional.values %}
            {% for subvalue in value %}
                <code class="data-item"><span>{{ subvalue }}</span></code><br>
            {% endfor %}
        {% endfor %}
        <br>

        <code>"nlc":<span> "{{ item.nlc }}"</span> </code>
        <br><br>
    </div>
{% endfor %}

{% endblock %}
