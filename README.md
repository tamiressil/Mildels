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
import json
from .models import TbMcastCluster, TbMcastDemarcation, TbMcastNacional

def get_city_data(cidade, uf):
    # Dados da tabela demarcation
    demarcation_data = TbMcastDemarcation.objects.filter(cidade=cidade, uf=uf).first()
    if not demarcation_data:
        return None
    
    demarcation_json = json.loads(json.dumps(demarcation_data.json_body)) if demarcation_data else {}  # Convertendo em um dicionário em python
    filtrando_demarcation_json = {}
    for key, value in demarcation_json.items():
        if key != 'sigla':
            filtrando_demarcation_json[key] = value

    # Dados da tabela do cluster
    todas_as_regioes = TbMcastCluster.objects.all()
    cluster_data = None
    for regiao in todas_as_regioes:
        regioes_list = {r.strip().lower() for r in regiao.region.split("_")}  # Pegando a lista de regiões e transformando em letras minúsculas e sem espaços extras.
        uf_formatada = uf.strip().lower()

        if uf_formatada in regioes_list:
            cluster_data = regiao
            break

    # Convertendo a tabela cluster em um dicionário para filtrar, e remover a chave 'region'
    cluster_json = json.loads(json.dumps(cluster_data.json_body)) if cluster_data else {}  # Convertendo em um dicionário em python
    filtrando_cluster_json = {}
    for key, value in cluster_json.items():
        if key != 'region':
            filtrando_cluster_json[key] = value

    # Tabela nacional, conversão para um dicionário com a filtragem para ver se ele é de SP ou de fora de SP, assim exibindo o json correto
    tipo_nacional = "dsp" if uf == "SP" else "fsp"
    nacional_data = TbMcastNacional.objects.filter(dsp_fsp=tipo_nacional).first()
    nacional_json = json.loads(json.dumps(nacional_data.json_body)) if nacional_data else {}  # Transformando ele em um dicionário

    filtrando_nacional_json = {}
    for key, value in nacional_json.items():
        if key == "channels":
            filtrando_nacional_json[key] = value

    # Envio dos dados.
    city_data = {
        "city": cidade,
        "cluster": (filtrando_cluster_json),  
        "demarcation": (filtrando_demarcation_json),  
        "federative_Unit": demarcation_data.uf.lower() if demarcation_data else None,
        "nacional": (filtrando_nacional_json),  
        "nlc": demarcation_data.sigla.lower() if demarcation_data else None,
    }
    return city_data

