<template>
    <div class="page">

        <form class="card" @submit.prevent="simular">
            <label>
                Preço do imóvel
                <input v-model.number="form.precoVenda" type="number" />
            </label>

            <label>
                Financiamento
                <input v-model.number="form.financiamentoSubsidio" type="number" />
            </label>

            <label>
                ATO
                <input v-model.number="form.valorAto" type="number" />
            </label>

            <label>
                Investimento inicial
                <input v-model.number="form.valorInvestimentoInicial" type="number" />
            </label>

            <label>
                Parcelas construtora
                <input v-model.number="form.numParcelasConstrutora" type="number" />
            </label>

            <label>
                Cenário
                <select v-model="form.cenarioEscolha">
                    <option value="investir">Investir</option>
                    <option value="ato">ATO adicional</option>
                </select>
            </label>

            <button type="submit">Simular</button>
        </form>

        <section v-if="resumo" class="card resumo">
            <h2>Resumo</h2>
            <p><strong>Entrada:</strong> {{ moeda(resumo.entrada_calculada) }}</p>
            <p><strong>Total INCC:</strong> {{ moeda(resumo.total_incc_pago) }}</p>
            <p><strong>Saldo final investimento:</strong> {{ moeda(resumo.saldo_final_investimento) }}</p>
        </section>

        <section v-if="detalhes.length" class="card">
            <h2>Detalhamento mensal</h2>

            <div class="table-scroll">
                <table>
                    <thead>
                        <tr>
                            <th v-for="(v, k) in detalhes[0]" :key="k">{{ k }}</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(linha, i) in detalhes" :key="i">
                            <td v-for="(v, k) in linha" :key="k">
                                {{ formatCell(v) }}
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </section>
    </div>
</template>
<script setup>
import { reactive, ref } from "vue";

const API_URL = import.meta.env.VITE_API_URL;



const form = reactive({
    precoVenda: 230000,
    financiamentoSubsidio: 150000,
    valorAto: 10000,
    valorInvestimentoInicial: 15000,
    numParcelasConstrutora: 6,
    cenarioEscolha: "investir"
});

const resumo = ref(null);
const detalhes = ref([]);

async function simular() {
    const res = await fetch(API_URL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(form)
    });

    const data = await res.json();
    resumo.value = data.resumo_final;
    detalhes.value = data.detalhes_mes_a_mes;
}

function moeda(v) {
    return new Intl.NumberFormat("pt-BR", {
        style: "currency",
        currency: "BRL"
    }).format(v || 0);
}

function formatCell(v) {
    return typeof v === "number" ? moeda(v) : v;
}
</script>
<style scoped>
/* ===== BASE ===== */
.page {
    max-width: 1200px;
    margin: 0 auto;
    padding: 16px;
    font-family: system-ui, Arial, sans-serif;
    color: #111;
}

h1 {
    margin-bottom: 16px;
}

/* ===== CARD ===== */
.card {
    background: #ffffff;
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 16px;
    margin-bottom: 16px;
}

/* ===== FORM ===== */
form label {
    display: block;
    margin-bottom: 12px;
    font-size: 14px;
}

input,
select {
    width: 100%;
    padding: 8px;
    margin-top: 4px;
    font-size: 14px;
    border: 1px solid #ccc;
    border-radius: 4px;
    background: #fff;
    color: #111;
}

button {
    width: 100%;
    padding: 10px;
    margin-top: 8px;
    background: #2563eb;
    color: #fff;
    font-weight: 600;
    border: none;
    border-radius: 6px;
    cursor: pointer;
}

button:hover {
    background: #1d4ed8;
}

/* ===== RESUMO ===== */
.resumo p {
    margin: 6px 0;
}

/* ===== TABELA ===== */
.table-scroll {
    width: 100%;
    overflow-x: auto;
}

table {
    width: max-content;
    min-width: 100%;
    border-collapse: collapse;
    font-size: 12px;
}

th,
td {
    border: 1px solid #ddd;
    padding: 6px;
    white-space: nowrap;
    text-align: right;
}

th {
    background: #f3f4f6;
    font-weight: 600;
}

tbody tr:nth-child(even) {
    background: #fafafa;
}
</style>
