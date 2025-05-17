{% extends 'base.html' %}
{% load static %}

{% block title %}Detalhes da Cidade{% endblock %}

{% block content %}
<link rel="stylesheet" href="{% static 'static_files/js/css/detalhes_cidade.css' %}">

{% for item in items %}
    <br>
    <h1>{{ item.city }}</h1>

    <div class="Quadrado_dados">
        <pre><code>{{ item | safe }}</code></pre>
    </div>
{% endfor %}

{% endblock %}
