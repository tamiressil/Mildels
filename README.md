{% for item in items %}
    <br>
    <h1>{{ item.city }}</h1>
    <br>
    <div class="Quadrado_dados">
        <br>
        <code>"city": {{ item.city }}</code><br>

        {% if item.cluster_filtrado %}
            {% for value in item.cluster_filtrado %}
                <code>{{ value }}</code><br>
            {% endfor %}
        {% else %}
            <code>Cluster não encontrado</code>
        {% endif %}  <!-- ✅ Correção: agora está fechando corretamente o bloco `if` -->

        <code>"demarcation": {{ item.demarcation }}</code><br>
        <code>"federative Unit": {{ item.federative_unit }}</code><br>
        <code>"nacional": {{ item.nacional }}</code><br>
        <code>"nlc": {{ item.nlc }}</code>
    </div>
{% endfor %} <!-- ✅ Correção: agora está fechando corretamente o loop `for` -->
