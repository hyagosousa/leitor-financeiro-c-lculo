<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Resumo PDF Positivos</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.min.js"></script>

<style>
body { font-family: Arial; background: #111; color: white; text-align: center; padding: 20px; }
h1 { color: #00ffcc; }
input { margin: 20px; padding: 10px; }
table { width: 95%; margin: 20px auto; border-collapse: collapse; }
th, td { border: 1px solid #fff; padding: 8px; text-align: right; }
th { background: #00aa88; color: #fff; }
td:first-child { text-align: left; }
td { background: #063; }
.maior { background: #0044ff; }
.menor { background: #880000; }
</style>
</head>

<body>

<h1>📄 Resumo de PDFs Positivos</h1>

<input type="file" id="pdfInput" multiple accept="application/pdf">

<table>
  <thead>
    <tr>
      <th>Arquivo</th>
      <th>Resultado (2600)</th>
      <th>Produtos (2603)</th>
      <th>Mercadoria (2652)</th>
      <th>Serviços (2700)</th>
      <th>Simples (2831)</th>
      <th>Diferença</th>
      <th>Comparação</th>
    </tr>
  </thead>
  <tbody id="tabelaResumo"></tbody>
</table>

<script>

// 🔧 IMPORTANTE (worker do PDF.js)
pdfjsLib.GlobalWorkerOptions.workerSrc =
  "https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.worker.min.js";

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

// 🔢 converter texto → número
function converterParaNumero(valor) {
  if (!valor || valor === "-") return 0;

  return parseFloat(
    valor
      .replace(/\./g, "")
      .replace(",", ".")
      .replace("(", "-")
      .replace(")", "")
  );
}

function extrairInformacoes(texto, nomeArquivo) {

  texto = texto.replace(/\s+/g, " ");

  function pegarUltimoValor(linha) {
    if (!linha) return "-";

    const numeros = linha.match(/\(?\d{1,3}(?:\.\d{3})*,\d{2}\)?/g);

    if (!numeros) return "-";

    return numeros[numeros.length - 1];
  }

  function buscarLinha(codigo) {
    const linhas = texto.split("\n");

    for (let linha of linhas) {
      if (linha.includes(codigo)) {
        return linha;
      }
    }
    return "";
  }

  const resultado = pegarUltimoValor(buscarLinha("2600"));
  const produtos = pegarUltimoValor(buscarLinha("2603"));
  const mercadoria = pegarUltimoValor(buscarLinha("2652"));
  const servicos = pegarUltimoValor(buscarLinha("2700"));
  const simples = pegarUltimoValor(buscarLinha("2831"));

  // 🔢 converter para número
  const vResultado = converterParaNumero(resultado);
  const vProdutos = converterParaNumero(produtos);
  const vMercadoria = converterParaNumero(mercadoria);
  const vServicos = converterParaNumero(servicos);
  const vSimples = converterParaNumero(simples);

  // 📊 cálculos
  const calcServicos = vServicos * 0.32;
  const calcSimples = vSimples * 0.05;

  const diferenca = calcServicos - calcSimples;

  // 🔍 comparação
  const comparacao = diferenca > vResultado ? "MAIOR" : "MENOR";

  const classe = comparacao === "MAIOR" ? "maior" : "menor";

  const tbody = document.getElementById("tabelaResumo");
  const tr = document.createElement("tr");

  tr.innerHTML = `
    <td>${nomeArquivo}</td>
    <td>${resultado}</td>
    <td>${produtos}</td>
    <td>${mercadoria}</td>
    <td>${servicos}</td>
    <td>${simples}</td>
    <td>${diferenca.toFixed(2)}</td>
    <td class="${classe}">${comparacao}</td>
  `;

  tbody.appendChild(tr);
}
</script>

</body>
</html>>
