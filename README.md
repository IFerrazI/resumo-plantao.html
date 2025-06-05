```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <title>Resumo do Plantão HRBA</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    body {
      font-family: Arial, sans-serif;
      font-size: 14px;
      margin: 20px;
      line-height: 1.4;
    }
    h1 {
      font-size: 18px;
      margin-bottom: 10px;
    }
    h2 {
      font-size: 16px;
      margin-top: 20px;
      margin-bottom: 5px;
      border-bottom: 1px solid #ccc;
      padding-bottom: 2px;
    }
    label {
      display: block;
      margin-top: 10px;
    }
    input[type="number"],
    input[type="text"],
    input[type="date"],
    select {
      width: 150px;
      padding: 4px;
      margin-left: 10px;
      font-size: 14px;
    }
    .btn {
      margin-top: 10px;
      padding: 6px 12px;
      font-size: 14px;
      cursor: pointer;
    }
    #listaAmarelos .item-amarelo,
    #listaAzuis .item-azul,
    #listaProcedimentos .item-procedimento {
      margin-top: 10px;
      padding: 8px;
      border: 1px solid #ddd;
      border-radius: 4px;
      background-color: #f9f9f9;
    }
    #listaAmarelos .item-amarelo label,
    #listaAzuis .item-azul label,
    #listaProcedimentos .item-procedimento label {
      display: inline-block;
      margin-right: 10px;
    }
    #listaAmarelos .item-amarelo input,
    #listaAmarelos .item-amarelo select,
    #listaAzuis .item-azul input,
    #listaAzuis .item-azul select,
    #listaProcedimentos .item-procedimento select,
    #listaProcedimentos .item-procedimento input {
      width: 140px;
    }
    textarea {
      margin-top: 20px;
      width: 100%;
      height: 220px;
      font-family: monospace;
      font-size: 13px;
      padding: 8px;
    }
    .small {
      font-size: 12px;
      color: #555;
    }
  </style>
</head>
<body>
  <h1>TRR HRBA - Produção</h1>

  <!-- Seção Data -->
  <label>
    Data:
    <input type="date" id="dataPicker" />
    <span class="small" style="margin-left: 8px;">(clique para alterar, se quiser usar data anterior)</span>
  </label>

  <!-- Seção Turno -->
  <label>
    Turno:
    <select id="turno">
      <option value="MANHÃ">MANHÃ</option>
      <option value="TARDE">TARDE</option>
      <option value="MANHÃ/TARDE">MANHÃ/TARDE</option>
    </select>
  </label>

  <!-- Seção Códigos -->
  <h2>Códigos</h2>

  <label>
    Códigos Branco:
    <input type="number" id="qtdBranco" min="0" value="0" />
  </label>

  <div style="margin-top: 15px;">
    <strong>Códigos Amarelo:</strong>
    <button type="button" class="btn" onclick="adicionarAmarelo()">➕ Adicionar Código Amarelo</button>
    <div id="listaAmarelos" class="small">
      <!-- Campos de Leito, Motivo e Desfecho para cada código amarelo -->
    </div>
  </div>

  <div style="margin-top: 15px;">
    <strong>Códigos Azul:</strong>
    <button type="button" class="btn" onclick="adicionarAzul()">➕ Adicionar Código Azul</button>
    <div id="listaAzuis" class="small">
      <!-- Campos de Leito + Desfecho para cada código azul -->
    </div>
  </div>

  <!-- Seção Prescrições entre Códigos e Transportes -->
  <h2>Prescrições UTI &gt; Enfermaria</h2>
  <label>
    Total Prescrições UTI &gt; Enfermaria:
    <input type="number" id="qtdPrescricoesUTI" min="0" value="0" />
  </label>

  <!-- Seção Transportes -->
  <h2>Transportes</h2>
  <label>
    Transportes Enfermaria &gt; UTI:
    <input type="number" id="qtdTransportes" min="0" value="0" />
  </label>

  <!-- Seção Procedimentos -->
  <h2>Procedimentos</h2>
  <button type="button" class="btn" onclick="adicionarProcedimento()">➕ Adicionar Procedimento</button>
  <div id="listaProcedimentos" class="small">
    <!-- Campos de Procedimento serão adicionados aqui -->
  </div>

  <!-- Botão para gerar a mensagem -->
  <button type="button" class="btn" onclick="gerarResumo()">Gerar mensagem WhatsApp</button>

  <!-- Textarea para conferência da mensagem -->
  <textarea id="txtResumo" readonly placeholder="Sua mensagem aparecerá aqui..."></textarea>

  <script>
    // Preenche o datepicker com a data de hoje ao carregar
    window.addEventListener('load', () => {
      const hoje = new Date();
      const ano = hoje.getFullYear();
      const mes = String(hoje.getMonth() + 1).padStart(2, '0');
      const dia = String(hoje.getDate()).padStart(2, '0');
      document.getElementById('dataPicker').value = `${ano}-${mes}-${dia}`;
    });

    // ================================
    // Códigos Amarelo Dinâmicos
    // ================================
    function adicionarAmarelo() {
      const container = document.getElementById('listaAmarelos');
      const div = document.createElement('div');
      div.className = 'item-amarelo';

      // Leito
      const labelLeito = document.createElement('label');
      labelLeito.innerText = 'Leito:';
      const inputLeito = document.createElement('input');
      inputLeito.type = 'text';
      inputLeito.placeholder = 'ex: Filhote 2';
      inputLeito.className = 'leito-amarelo';
      labelLeito.appendChild(inputLeito);

      // Motivo
      const labelMotivo = document.createElement('label');
      labelMotivo.innerText = 'Motivo:';
      const inputMotivo = document.createElement('input');
      inputMotivo.type = 'text';
      inputMotivo.placeholder = 'ex: crise convulsiva';
      inputMotivo.className = 'motivo-amarelo';
      labelMotivo.appendChild(inputMotivo);

      // Desfecho
      const labelDesfecho = document.createElement('label');
      labelDesfecho.innerText = 'Desfecho:';
      const selectDesfecho = document.createElement('select');
      selectDesfecho.className = 'desfecho-amarelo';
      ['Revertido', 'Evolução para código azul'].forEach(op => {
        const opt = document.createElement('option');
        opt.value = op;
        opt.innerText = op;
        selectDesfecho.appendChild(opt);
      });
      labelDesfecho.appendChild(selectDesfecho);

      // Botão Remover
      const btnRemover = document.createElement('button');
      btnRemover.type = 'button';
      btnRemover.innerText = '🗑️';
      btnRemover.style.marginLeft = '10px';
      btnRemover.onclick = () => container.removeChild(div);

      // Monta o bloco
      div.appendChild(labelLeito);
      div.appendChild(labelMotivo);
      div.appendChild(labelDesfecho);
      div.appendChild(btnRemover);
      container.appendChild(div);
    }

    // ================================
    // Códigos Azul Dinâmicos
    // ================================
    function adicionarAzul() {
      const container = document.getElementById('listaAzuis');
      const div = document.createElement('div');
      div.className = 'item-azul';

      // Leito
      const labelLeito = document.createElement('label');
      labelLeito.innerText = 'Leito:';
      const inputLeito = document.createElement('input');
      inputLeito.type = 'text';
      inputLeito.placeholder = 'ex: Piratininga 3';
      inputLeito.className = 'leito-azul';
      labelLeito.appendChild(inputLeito);

      // Desfecho
      const labelDesfecho = document.createElement('label');
      labelDesfecho.innerText = 'Desfecho:';
      const selectDesfecho = document.createElement('select');
      selectDesfecho.className = 'desfecho-azul';
      ['RCE', 'Óbito'].forEach(op => {
        const opt = document.createElement('option');
        opt.value = op;
        opt.innerText = op;
        selectDesfecho.appendChild(opt);
      });
      labelDesfecho.appendChild(selectDesfecho);

      // Botão Remover
      const btnRemover = document.createElement('button');
      btnRemover.type = 'button';
      btnRemover.innerText = '🗑️';
      btnRemover.style.marginLeft = '10px';
      btnRemover.onclick = () => container.removeChild(div);

      // Monta o bloco
      div.appendChild(labelLeito);
      div.appendChild(labelDesfecho);
      div.appendChild(btnRemover);
      container.appendChild(div);
    }

    // ================================
    // Procedimentos Dinâmicos
    // ================================
    function adicionarProcedimento() {
      const container = document.getElementById('listaProcedimentos');
      const div = document.createElement('div');
      div.className = 'item-procedimento';

      const labelTipo = document.createElement('label');
      labelTipo.innerText = 'Procedimento:';
      const selectTipo = document.createElement('select');
      selectTipo.className = 'tipo-procedimento';
      ['Catéter venoso central',
       'Intubação orotraqueal',
       'Paracentese',
       'Toracocentese',
       'Outros'
      ].forEach(op => {
        const opt = document.createElement('option');
        opt.value = op;
        opt.innerText = op;
        selectTipo.appendChild(opt);
      });
      labelTipo.appendChild(selectTipo);

      const inputOutro = document.createElement('input');
      inputOutro.type = 'text';
      inputOutro.placeholder = 'Descreva se “Outros”';
      inputOutro.className = 'outro-procedimento';
      inputOutro.style.display = 'none';

      selectTipo.onchange = () => {
        inputOutro.style.display = selectTipo.value === 'Outros' ? 'inline-block' : 'none';
        if (selectTipo.value !== 'Outros') inputOutro.value = '';
      };

      const btnRemover = document.createElement('button');
      btnRemover.type = 'button';
      btnRemover.innerText = '🗑️';
      btnRemover.style.marginLeft = '10px';
      btnRemover.onclick = () => container.removeChild(div);

      div.appendChild(labelTipo);
      div.appendChild(inputOutro);
      div.appendChild(btnRemover);
      container.appendChild(div);
    }

    // ================================
    // Auxiliar de Data
    // ================================
    function obterDiaMes() {
      const valor = document.getElementById('dataPicker').value; // "YYYY-MM-DD"
      if (!valor) {
        const hoje = new Date();
        const d = String(hoje.getDate()).padStart(2, '0');
        const m = String(hoje.getMonth() + 1).padStart(2, '0');
        return { dia: d, mes: m };
      }
      const partes = valor.split('-'); // ["YYYY","MM","DD"]
      return { dia: partes[2], mes: partes[1] };
    }

    // ================================
    // Montagem da Mensagem
    // ================================
    function gerarResumo() {
      // 1. Códigos Branco
      const branco = parseInt(document.getElementById('qtdBranco').value) || 0;

      // 2. Códigos Amarelo
      const listaAmarelos = document.querySelectorAll('#listaAmarelos .item-amarelo');
      const amarelosCount = listaAmarelos.length;
      const detalhesAmarelos = [];
      listaAmarelos.forEach(item => {
        const leito = item.querySelector('.leito-amarelo').value.trim() || '–';
        const motivo = item.querySelector('.motivo-amarelo').value.trim() || '–';
        const desfecho = item.querySelector('.desfecho-amarelo').value;
        detalhesAmarelos.push({ leito, motivo, desfecho });
      });

      // 3. Códigos Azul
      const listaAzuis = document.querySelectorAll('#listaAzuis .item-azul');
      const azuisCount = listaAzuis.length;
      const detalhesAzuis = [];
      listaAzuis.forEach(item => {
        const leito = item.querySelector('.leito-azul').value.trim() || '–';
        const desfecho = item.querySelector('.desfecho-azul').value;
        detalhesAzuis.push({ leito, desfecho });
      });

      // 4. Prescrições UTI > Enfermaria
      const prescricoesUTI = parseInt(document.getElementById('qtdPrescricoesUTI').value) || 0;

      // 5. Transportes
      const trans = parseInt(document.getElementById('qtdTransportes').value) || 0;

      // 6. Procedimentos
      const listaProceds = document.querySelectorAll('#listaProcedimentos .item-procedimento');
      const procedimentosCount = listaProceds.length;
      const detalhesProceds = [];
      listaProceds.forEach(item => {
        const tipo = item.querySelector('.tipo-procedimento').value;
        let descricao = tipo;
        if (tipo === 'Outros') {
          const outroText = item.querySelector('.outro-procedimento').value.trim() || '–';
          descricao = `Outros: ${outroText}`;
        }
        detalhesProceds.push(descricao);
      });

      // 7. Data e Turno
      const { dia, mes } = obterDiaMes();
      const turno = document.getElementById('turno').value;

      // Montagem da mensagem
      let mensagem = `*TRR HRBA - ${dia}/${mes} - ${turno}*%0A%0A`;

      // Seção Códigos
      mensagem += `*Códigos:*%0A`;
      mensagem += `• Branco: ${branco}%0A`;
      mensagem += `• Amarelo (${amarelosCount}):%0A`;
      if (amarelosCount > 0) {
        detalhesAmarelos.forEach(({ leito, motivo, desfecho }) => {
          mensagem += `  – Leito: ${leito}, Motivo: ${motivo}, Desfecho: ${desfecho}%0A`;
        });
      } else {
        mensagem += `  – (Nenhum código amarelo registrado)%0A`;
      }
      mensagem += `• Azul (${azuisCount}):%0A`;
      if (azuisCount > 0) {
        detalhesAzuis.forEach(({ leito, desfecho }) => {
          mensagem += `  – Leito: ${leito}, Desfecho: ${desfecho}%0A`;
        });
      } else {
        mensagem += `  – (Nenhum código azul registrado)%0A`;
      }
      mensagem += `%0A`;

      // Seção Prescrições UTI > Enfermaria
      mensagem += `*Prescrições UTI > Enfermaria:* ${prescricoesUTI}%0A%0A`;

      // Seção Transportes
      mensagem += `*Transportes Enfermaria > UTI:* ${trans}%0A%0A`;

      // Seção Procedimentos
      mensagem += `*Procedimentos (${procedimentosCount}):*%0A`;
      if (procedimentosCount > 0) {
        detalhesProceds.forEach(desc => {
          mensagem += `  – ${desc}%0A`;
        });
      } else {
        mensagem += `  – (Nenhum procedimento registrado)%0A`;
      }

      // Texto “desformatado” para a textarea
      const textoPlano = mensagem.replace(/%0A/g, '\n');
      document.getElementById('txtResumo').value = textoPlano;

      // Abre WhatsApp Web/App com a mensagem formatada
      const urlWhatsapp = `https://wa.me/?text=${mensagem}`;
      window.open(urlWhatsapp, '_blank');
    }
  </script>
</body>
</html>
```
