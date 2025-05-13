{% block content %}
    {% for item in items %}
        <div class="Quadrado_dados">
            <h1>{{ item.city }}</h1>
            {% for key, value in item.cluster.items %}
                <div class="dados_linha">
                    <strong>{{ key }}:</strong> {{ value }}
                </div>
            {% endfor %}
        </div>
    {% endfor %}  <!-- ✅ Certifique-se de que este `endfor` existe -->
{% endblock %}
