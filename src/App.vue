<script setup>
// ref cria referencia reativa, shallowRef atualiza apenas uma parte, computed cria um valor computado reativo, nextTick atualiza com base na alteração dos dados
import { ref, shallowRef, computed, watch, nextTick } from 'vue'
// biblioteca para criar gráficos e visualizar dados
import Chart from 'chart.js/auto'

//referencia reativa para um array vazio
const weights = ref([])

//referencia reativa para um valor inicial nulo.
const weightChartEl = ref(null)

// referencia reativa rasa, armazena uma instancia do objeto Chart.js
const weightChart = shallowRef(null)

//referencia reativa com valor inicial zero
const weightInput = ref(0)

//variável computada para calcular o peso atual do usuário
const currentWeight = computed(() => {
	//classifica o array em ordem decrescente, sendo primeiro o mais recente
	return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 }
})

//adicionar um novo peso ao array de pesos armazenados em 'weights'
const addWeight = () => {
	weights.value.push({
		//recebe o valor atual 'weightInput' do usuário para o novo peso a ser adicionado
		weight: weightInput.value,
		// data atual
		date: new Date().getTime()
	})
}

// mudanças na referencia reativa 'weights'
watch(weights, (newWeights) => {
	// cria uma cópia do array sem modificar o array original
	const ws = [...newWeights]

	//valor atual da referencia 'weightChart'existir
	if (weightChart.value) {
		// dados dos gráficos
		weightChart.value.data.labels = ws
			//ordenação das datas do array
			.sort((a, b) => a.date - b.date)
			//transforma cada objeto de 'ws' em uma string de data formatada
			.map(weight => new Date(weight.date).toLocaleDateString() )
			// exibe os dados dos últimos 7 dias
			.slice(-7)

		//atualiza os dados do gráfico de peso
		weightChart.value.data.datasets[0].data = ws
			//ordenação das datas do array
			.sort((a, b) => a.date - b.date)
			//extrai o peso de cada objeto em 'ws'
			.map(weight => weight.weight)
			//retorna os últimos 7 pesos do array
			.slice(-7)

		//atualiza o gráfico de peso com os novos dados que foram definidos
		weightChart.value.update()
		return
	}

	//gráfico seja criado apenas após a atualização do valor de referencia 'weightChartEl'
	nextTick(() => {
		// cria um novo gráfico em 2d
		weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
			//tipo de gráfico: linhas
			type: 'line',
			// dados a serem exibidos
			data: {
				labels: ws
					//ordenação das datas do array
					.sort((a, b) => a.date - b.date)
					//transforma cada objeto de 'ws' em uma string de data formatada
					.map(weight => new Date(weight.date).toLocaleDateString()),
				datasets: [
					{
						label: 'Peso',
						data: ws
							//ordenação das datas do array
							.sort((a, b) => a.date - b.date)
							//extrai o peso de cada objeto em 'ws'
							.map(weight => weight.weight),
						// cor abaixo da linha do gráfico
						backgroundColor: 'rgba(255, 105, 180, 0.2)',
						// a cor da linha do gráfico
						borderColor: 'rgba(255, 105, 180, 1)',
						// largura da linha
						borderWidth: 1,
						// área abaixo da linha será preenchida com a cor de fundo
						fill: true
					}
				]
			},
			options: {
				//permite que o gráfico seja redimensionado de forma responsiva
				responsive: true,
				//o gráfico pode ser redimensionado livremente em todas as direções
				maintainAspectRatio: false
			}
		})
	})
	//alterações de propriedades profundas serão monitoradas, se houver mudanças 
	//dentro do objeto 'weights' elas serão detectadas pelo 'watch'
}, { deep: true })
</script>

<template>
	<main>

		<h1>Acompanhamento de peso</h1>

		<div class="current">
			<span>{{ currentWeight.weight }}</span>

			<small>Peso atual (kg)</small>

		</div>

		<!--previne o comportamento padrão de envio do formulário (ou seja, a atualização da página). 
		Quando o usuário clicar no botão de enviar dentro do formulário, a função addWeight será executada.-->
		<form @submit.prevent="addWeight">
			<!-- step="0.1" é o intervalo sugerido entre os valores numéricos para um controle de entrada numérica.-->
			<!-- Qualquer alteração no valor do elemento de formulário é refletida na variável weightInput e 
				qualquer alteração na variável weightInput é refletida no valor do elemento de formulário.-->
			<input 
				type="number"
				step="0.1" 
				v-model="weightInput" />

			<input	
				type="submit"
				value="Adicionar peso" />
		</form>
		
		<!--se a variável weights existir e seu comprimento for maior que 0,
			o conteúdo dentro da div será renderizado.-->
		<div v-if="weights && weights.length > 0">

			<h2>
				Últimos 7 dias
			</h2>
			
			<!--canvas é usado para renderizar o gráfico de pesos e referenciado pelo atributo ref-->
			<div class="canvas-box">
				<canvas ref="weightChartEl"></canvas>
			</div>

			<div class="weight-history">

				<h2>Histórico de pesos</h2>

				<ul>
					<!--iteração sobre o array 'weights' exibindo o peso e a data 
					de cada objeto 'weight' do array-->
					<!--A propriedade :key="weight.date" é usada para dar uma chave única a cada item da lista, 
						permitindo ao Vue.js efetivamente rastrear e atualizar o DOM quando os dados mudam.-->
					<li v-for="weight in weights" :key="weight.date">
						<span>{{ weight.weight }} kg</span>
						<small>
							{{ new Date(weight.date).toLocaleDateString() }}
						</small>
					</li>
				</ul>
			</div>

		</div>

	</main>
</template>

<style>
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	font-family: 'montserrat', sans-serif;
}

body {
	background-color: #eee;
}

main {
	padding: 1.5rem;
}

h1 {
	font-size: 2em;
	text-align: center;
	margin-bottom: 2rem;
}

h2 {
	margin-bottom: 1rem;
	color: #0213fd;
	font-weight: 400;
}

.current {
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;

	width: 200px;
	height: 200px;
	
	text-align: center;
	background-color: white;
	border-radius: 999px;
	box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
	border: 5px solid hwb(231 0% 3%);
	
	margin: 0 auto 2rem;
}

.current span {
	display: block;
	font-size: 2em;
	font-weight: bold;
	margin-bottom: 0.5rem;
}

.current small {
	color: hwb(231 0% 3%);
	font-style: italic;
}

form {
	display: flex;
	margin-bottom: 2rem;
	border: 1px solid #AAA;
	border-radius: 0.5rem;
	overflow: hidden;
	transition: 200ms linear;
}

form:focus-within,
form:hover {
	border-color: hwb(231 0% 3%);
	border-width: 2px;
}

form input[type="number"] {
	appearance: none;
	outline: none;
	border: none;
	background-color: white;

	flex: 1 1 0%;
	padding: 1rem 1.5rem;
	font-size: 1.25rem;
}

form input[type="submit"] {
	appearance: none;
	outline: none;
	border: none;
	cursor: pointer;
	background-color: hwb(231 0% 3%);

	padding: 0.5rem 1rem;

	color: white;
	font-size: 1.25rem;
	font-weight: 700;
	transition: 200ms linear;
	border-left: 3px solid transparent;
}

form input[type="submit"]:hover {
	background-color: white;
	color: hwb(231 0% 3%);;
	border-left-color: hwb(231 0% 3%);
}

.canvas-box {
	width: 100%;
	max-width: 720px;
	background-color: rgb(255, 255, 255);
	padding: 1rem;
	border-radius: 0.5rem;
	box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
	margin-bottom: 2rem;
}

.weight-history ul {
	list-style: none;
	padding: 0;
	margin: 0;
}

.weight-history ul li {
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 0.5rem;
	cursor: pointer;
}

.weight-history ul li:nth-child(even) {
	background-color: #dfdfdf;
}

.weight-history ul li:hover {
	background-color: #f8f8f8;
}

.weight-history ul li:last-of-type {
	border-bottom: none;
}

.weight-history ul li span {
	display: block;
	font-size: 1.25rem;
	font-weight: 700;
	margin-right: 1rem;
}

.weight-history ul li small {
	color: hwb(231 0% 3%);
	font-style: italic;
}
</style>
