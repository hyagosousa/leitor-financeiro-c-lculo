<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Resumo PDF Positivos com Cálculos</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.min.js"></script>

<style>
body { font-family: Arial; background: #111; color: white; text-align: center; padding: 20px; }
h1 { color: #00ffcc; }
input { margin: 20px; padding: 10px; }
table { width: 98%; margin: 20px auto; border-collapse: collapse; }
th, td { border: 1px solid #fff; padding: 8px; text-align: right; }
th { background: #00aa88; color: #fff; }
td:first-child { text-align: left; }
td { background: #063; }
</style>
</head>

<body>

<h1>📄 Resumo de PDFs Positivos com Cálculos</h1>

<input type="file" id="pdfInput" multiple accept="application/pdf">

<table>
  <thead>
    <tr>
      <th>Arquivo</th>
      <th>Resultado do Período (2600)</th>
      <th>Produto (2603)</th>
      <th>Produto × 8%</th>
      <th>Mercadoria (2652)</th>
      <th>Mercadoria × 8%</th>
      <th>Prestação Serviços (2700)</th>
      <th>Prestação × 32%</th>
      <th>Simples Nacional (2831)</th>
      <th>Simples × 5%</th>
      <th>Diferença (Prestação×32% − Simples×5%)</th>
      <th>Comparação</th>
    </tr>
  </thead>
  <tbody id="tabelaResumo"></tbody>
</table>

<script>
const input = document.getElementById("pdfInput");

input.addEventListener("change", async (event) => {
  document.getElementById("tabelaResumo").innerHTML = "";
  const arquivos = event.target.files;

  for (let file of arquivos) {
    const texto = await lerPDF(file);
    extrairInformacoes(texto, file.name);
  }
});

async function lerPDF(file) {
  const reader = new FileReader();
  return new Promise((resolve) => {
    reader.onload = async function () {
      const typedarray = new Uint8Array(this.result);
      const pdf = await pdfjsLib.getDocument(typedarray).promise;
      let texto = "";
      for (let i = 1; i <= pdf.numPages; i++) {
        const pagina = await pdf.getPage(i);
        const conteudo = await pagina.getTextContent();
        conteudo.items.forEach(item => { texto += item.str + " "; });
        texto += "\n";
      }
      resolve(texto.toLowerCase());
    };
    reader.readAsArrayBuffer(file);
  });
}

function extrairInformacoes(texto, nomeArquivo) {

  texto = texto.replace(/\s+/g, " ");

  function pegarSaldoDaLinha(linha) {
    if (!linha) return 0;
    const numeros = linha.match(/\(?\d{1,3}(?:\.\d{3})*,\d{2}\)?/g);
    if (!numeros || numeros.length < 4) return 0;
    let saldo = numeros[3].replace(/\./g,"").replace(",","."); // transforma em número
    saldo = saldo.replace("(","-").replace(")",""); // trata negativos
    return parseFloat(saldo);
  }

  function buscarLinha(codigo) {
    const regex = new RegExp(`${codigo}.*?(?=\\d{4}|$)`, "i");
    const match = texto.match(regex);
    return match ? match[0] : "";
  }

  const resultadoLinha = buscarLinha("2600");
  const produtoLinha = buscarLinha("2603");
  const mercadoriaLinha = buscarLinha("2652");
  const servicosLinha = buscarLinha("2700");
  const simplesLinha = buscarLinha("2831");

  const resultado = pegarSaldoDaLinha(resultadoLinha);
  const produto = pegarSaldoDaLinha(produtoLinha);
  const mercadoria = pegarSaldoDaLinha(mercadoriaLinha);
  const prestacao = pegarSaldoDaLinha(servicosLinha);
  const simples = pegarSaldoDaLinha(simplesLinha);

  const produtoCalc = produto * 0.08;
  const mercadoriaCalc = mercadoria * 0.08;
  const prestacaoCalc = prestacao * 0.32;
  const simplesCalc = simples * 0.05;
  const diferenca = prestacaoCalc - simplesCalc;
  const comparacao = diferenca > resultado ? "MAIOR" : "MENOR";

  const tbody = document.getElementById("tabelaResumo");
  const tr = document.createElement("tr");
  tr.innerHTML = `
    <td>${nomeArquivo}</td>
    <td>${resultado.toLocaleString('pt-BR',{minimumFractionDigits:2, maximumFractionDigits:2})}</td>
    <td>${produto.toLocaleString('pt-BR',{minimumFractionDigits:2, maximumFractionDigits:2})}</td>
    <td>${produtoCalc.toLocaleString('pt-BR',{minimumFractionDigits:2, maximumFractionDigits:2})}</td>
    <td>${mercadoria.toLocaleString('pt-BR',{minimumFractionDigits:2, maximumFractionDigits:2})}</td>
    <td>${mercadoriaCalc.toLocaleString('pt-BR',{minimumFractionDigits:2, maximumFractionDigits:2})}</td>
    <td>${prestacao.toLocaleString('pt-BR',{minimumFractionDigits:2, maximumFractionDigits:2})}</td>
    <td>${prestacaoCalc.toLocaleString('pt-BR',{minimumFractionDigits:2, maximumFractionDigits:2})}</td>
    <td>${simples.toLocaleString('pt-BR',{minimumFractionDigits:2, maximumFractionDigits:2})}</td>
    <td>${simplesCalc.toLocaleString('pt-BR',{minimumFractionDigits:2, maximumFractionDigits:2})}</td>
    <td>${diferenca.toLocaleString('pt-BR',{minimumFractionDigits:2, maximumFractionDigits:2})}</td>
    <td>${comparacao}</td>
  `;
  tbody.appendChild(tr);
}
</script>

</body>
</html>
