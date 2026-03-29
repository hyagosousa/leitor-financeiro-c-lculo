<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Calculadora de Balancete Positivos</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.min.js"></script>

<style>
body { font-family: Arial; background:#111; color:white; padding:20px; text-align:center;}
h1 { color:#00ffcc; }
input { margin:20px; padding:10px; }
table { width: 100%; border-collapse: collapse; margin-top: 20px; color:white; }
th, td { border:1px solid #888; padding:8px; text-align:right; }
th { background:#333; }
tr:nth-child(even){background:#222;}
</style>
</head>
<body>

<h1>📊 Calculadora de Balancete - PDFs Positivos</h1>

<input type="file" id="pdfInput" multiple accept="application/pdf">
<br>

<table id="tabelaResultados">
  <thead>
    <tr>
      <th>Arquivo</th>
      <th>2603 Produto ×8%</th>
      <th>2652 Mercadoria ×8%</th>
      <th>2700 Prestação ×32%</th>
      <th>2831 Simples ×5%</th>
      <th>Resultado Calc</th>
      <th>Resultado Período</th>
      <th>Comparação</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<script>
const input = document.getElementById("pdfInput");

input.addEventListener("change", async (event) => {
  const arquivos = event.target.files;
  const tbody = document.querySelector("#tabelaResultados tbody");
  tbody.innerHTML = "";

  for (let file of arquivos) {
    const texto = await lerPDF(file);
    processarPDF(file.name, texto);
  }
});

async function lerPDF(file) {
  const reader = new FileReader();
  return new Promise((resolve) => {
    reader.onload = async function() {
      const typedarray = new Uint8Array(this.result);
      const pdf = await pdfjsLib.getDocument(typedarray).promise;
      let texto = "";
      for(let i=1;i<=pdf.numPages;i++){
        const page = await pdf.getPage(i);
        const content = await page.getTextContent();
        content.items.forEach(item => texto += item.str + " ");
        texto += "\n";
      }
      resolve(texto.toLowerCase());
    };
    reader.readAsArrayBuffer(file);
  });
}

function extrairValorFlex(texto, codigo){
  // Procura a linha que começa com o código
  const regexLinha = new RegExp(codigo + "\\s+[^\\n]+", "i");
  const match = texto.match(regexLinha);
  if(match){
    // Pega todos os números na linha
    const numeros = match[0].match(/\(?\d{1,3}(?:\.\d{3})*,\d{2}\)?/g);
    if(numeros && numeros.length > 0){
      // Pega o último número (saldo)
      let valor = numeros[numeros.length -1];
      valor = valor.replace(/[()]/g,'').replace(/\./g,'').replace(',','.');
      return parseFloat(valor);
    }
  }
  return 0;
}

function processarPDF(nomeArquivo, texto){
  const produto = extrairValorFlex(texto,"2603") || 0;
  const mercadoria = extrairValorFlex(texto,"2652") || 0;
  const prestacao = extrairValorFlex(texto,"2700") || 0;
  const simples = extrairValorFlex(texto,"2831") || 0;
  const resultadoPeriodo = extrairValorFlex(texto,"2600") || 0;

  const produtoCalc = produto * 0.08;
  const mercadoriaCalc = mercadoria * 0.08;
  const prestacaoCalc = prestacao * 0.32;
  const simplesCalc = simples * 0.05;

  const resultadoCalc = (produtoCalc + mercadoriaCalc + prestacaoCalc) - simplesCalc;

  const comparacao = resultadoCalc >= resultadoPeriodo ? "MAIOR" : "MENOR";

  const tbody = document.querySelector("#tabelaResultados tbody");
  const tr = document.createElement("tr");
  tr.innerHTML = `
    <td style="text-align:left">${nomeArquivo}</td>
    <td>${produtoCalc.toFixed(2)}</td>
    <td>${mercadoriaCalc.toFixed(2)}</td>
    <td>${prestacaoCalc.toFixed(2)}</td>
    <td>${simplesCalc.toFixed(2)}</td>
    <td>${resultadoCalc.toFixed(2)}</td>
    <td>${resultadoPeriodo.toFixed(2)}</td>
    <td>${comparacao}</td>
  `;
  tbody.appendChild(tr);
}
</script>

</body>
</html>
