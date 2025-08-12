<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Allenamenti - Calcolo Passo</title>
<style>
  body {
    font-family: Arial, sans-serif;
    background: #e0f0ff;
    color: #003366;
    margin: 0; padding: 20px;
  }
  h1 {
    text-align: center;
    color: #004a99;
  }
  form {
    background: #cce0ff;
    padding: 15px;
    border-radius: 8px;
    max-width: 400px;
    margin: auto;
  }
  label {
    display: block;
    margin: 12px 0 4px;
  }
  input, select {
    width: 100%;
    padding: 6px;
    font-size: 16px;
    border-radius: 4px;
    border: 1px solid #004a99;
  }
  button {
    margin-top: 15px;
    width: 100%;
    background: #004a99;
    color: white;
    font-size: 18px;
    padding: 10px;
    border: none;
    border-radius: 6px;
    cursor: pointer;
  }
  button:hover {
    background: #003366;
  }
  #result {
    max-width: 400px;
    margin: 20px auto;
    background: #b3d1ff;
    padding: 15px;
    border-radius: 8px;
    font-size: 18px;
  }
  #trend {
    max-width: 600px;
    margin: 30px auto;
    background: #cce6ff;
    border-radius: 8px;
    padding: 15px;
  }
  canvas {
    width: 100% !important;
    height: 300px !important;
  }
  /* Nuova schermata lista allenamenti */
  #allWorkoutsScreen {
    max-width: 800px;
    margin: 20px auto;
    background: #cce6ff;
    padding: 15px;
    border-radius: 8px;
    display: none;
  }
  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 15px;
  }
  th, td {
    border: 1px solid #004a99;
    padding: 8px;
    text-align: center;
  }
  th {
    background-color: #004a99;
    color: white;
  }
  #showListBtn {
    max-width: 400px;
    margin: 15px auto;
    display: block;
    background: #0066cc;
  }
  .deleteBtn {
    background: #cc3300;
    border: none;
    padding: 6px 12px;
    color: white;
    border-radius: 4px;
    cursor: pointer;
  }
  .deleteBtn:hover {
    background: #990000;
  }
</style>
</head>
<body>

<h1>Allenamenti - Calcolo Passo</h1>

<!-- Bottone per mostrare la lista allenamenti -->
<button id="showListBtn">Visualizza Tutti gli Allenamenti</button>

<!-- Schermata principale -->
<div id="mainScreen">
  <form id="form">
    <label for="date">Data</label>
    <input type="date" id="date" required />

    <label for="timeofday">Orario</label>
    <select id="timeofday" required>
      <option value="">Seleziona...</option>
      <option value="Mattina">Mattina</option>
      <option value="Sera">Sera</option>
    </select>

    <label for="distance">Percorso (km)</label>
    <input type="number" id="distance" min="0" step="0.01" placeholder="es. 5.00" required />

    <label for="time">Tempo (min:ss)</label>
    <input type="text" id="time" placeholder="es. 23:45" pattern="^[0-9]+:[0-5][0-9]$" required />

    <button type="submit">Calcola Passo e Salva</button>
  </form>

  <div id="result"></div>

  <div id="trend">
    <h2>Visualizza Trend Passo</h2>
    <label for="startdate">Da:</label>
    <input type="date" id="startdate" />
    <label for="enddate">A:</label>
    <input type="date" id="enddate" />
    <button id="showTrend">Mostra Trend</button>
    <canvas id="trendChart"></canvas>
  </div>
</div>

<!-- Schermata lista allenamenti -->
<div id="allWorkoutsScreen">
  <h2>Tutti gli Allenamenti Salvati</h2>
  <button id="backBtn">← Torna alla schermata principale</button>
  <table>
    <thead>
      <tr>
        <th>Data</th>
        <th>Orario</th>
        <th>Distanza (km)</th>
        <th>Tempo</th>
        <th>Passo (min/km)</th>
        <th>Azioni</th>
      </tr>
    </thead>
    <tbody id="workoutsTableBody">
      <!-- Allenamenti caricati qui -->
    </tbody>
  </table>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  // Funzione per convertire min:ss in secondi
  function timeToSeconds(t) {
    const [min, sec] = t.split(':').map(Number);
    return min * 60 + sec;
  }
  // Funzione per convertire secondi in min:ss
  function secondsToMinSec(s) {
    const min = Math.floor(s / 60);
    const sec = s % 60;
    return min + ':' + (sec < 10 ? '0' + sec : sec);
  }

  // Calcolo passo in min/km
  function calcPace(timeStr, dist) {
    const totalSeconds = timeToSeconds(timeStr);
    if(dist === 0) return 'N/A';
    const paceSeconds = totalSeconds / dist;
    return secondsToMinSec(Math.round(paceSeconds));
  }

  // Salvataggio dati in localStorage
  function saveData(entry) {
    let data = JSON.parse(localStorage.getItem('allenamenti')) || [];
    data.push(entry);
    localStorage.setItem('allenamenti', JSON.stringify(data));
  }

  // Recupera dati da localStorage
  function getData() {
    return JSON.parse(localStorage.getItem('allenamenti')) || [];
  }

  // Funzione per mostrare la lista allenamenti in tabella con pulsante elimina
  function renderWorkoutList() {
    const data = getData();
    const tbody = document.getElementById('workoutsTableBody');
    tbody.innerHTML = '';
    if(data.length === 0) {
      tbody.innerHTML = `<tr><td colspan="6">Nessun allenamento salvato.</td></tr>`;
      return;
    }
    // Ordina per data decrescente (più recenti in cima)
    data.sort((a,b) => b.date.localeCompare(a.date));
    data.forEach((d, index) => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${d.date}</td>
        <td>${d.timeofday}</td>
        <td>${d.distance.toFixed(2)}</td>
        <td>${d.time}</td>
        <td>${d.pace}</td>
        <td><button class="deleteBtn" data-index="${index}">Elimina</button></td>
      `;
      tbody.appendChild(tr);
    });

    // Aggiungi event listener ai pulsanti elimina
    document.querySelectorAll('.deleteBtn').forEach(btn => {
      btn.addEventListener('click', e => {
        const idx = e.target.getAttribute('data-index');
        deleteWorkout(parseInt(idx));
      });
    });
  }

  // Funzione per eliminare allenamento da localStorage
  function deleteWorkout(index) {
    let data = getData();
    data.splice(index, 1); // rimuovi elemento
    localStorage.setItem('allenamenti', JSON.stringify(data));
    renderWorkoutList(); // aggiorna tabella
  }

  // Gestione submit form
  document.getElementById('form').addEventListener('submit', e => {
    e.preventDefault();
    const date = document.getElementById('date').value;
    const timeofday = document.getElementById('timeofday').value;
    const distance = parseFloat(document.getElementById('distance').value);
    const time = document.getElementById('time').value;

    if (!date || !timeofday || isNaN(distance) || !time) {
      alert('Compila tutti i campi correttamente.');
      return;
    }

    // Calcolo passo
    const pace = calcPace(time, distance);

    // Salva
    saveData({ date, timeofday, distance, time, pace });

    // Mostra risultato
    document.getElementById('result').innerHTML = 
      `<strong>Passo calcolato:</strong> ${pace} min/km<br>
      Allenamento salvato per il ${date}.`;

    // Reset form
    e.target.reset();
  });

  // Grafico trend
  let chart = null;
  document.getElementById('showTrend').addEventListener('click', () => {
    const start = document.getElementById('startdate').value;
    const end = document.getElementById('enddate').value;
    if (!start || !end) {
      alert('Seleziona entrambe le date per visualizzare il trend.');
      return;
    }
    if (start > end) {
      alert('La data di inizio deve essere precedente alla data di fine.');
      return;
    }

    let data = getData();

    // Filtra per intervallo date
    data = data.filter(d => d.date >= start && d.date <= end);

    if(data.length === 0){
      alert('Nessun allenamento in questo intervallo.');
      return;
    }

    // Ordina per data
    data.sort((a,b) => a.date.localeCompare(b.date));

    // Prepara dati per il grafico
    const labels = data.map(d => d.date);
    const pacesInSeconds = data.map(d => {
      const [m,s] = d.pace.split(':').map(Number);
      return m*60 + s;
    });

    // Converti passo in min/km per etichette (ma chart usa secondi)
    const paceLabels = data.map(d => d.pace);

    // Crea o aggiorna grafico
    if(chart) chart.destroy();

    const ctx = document.getElementById('trendChart').getContext('2d');
    chart = new Chart(ctx, {
      type: 'line',
      data: {
        labels,
        datasets: [{
          label: 'Passo (min/km)',
          data: pacesInSeconds,
          borderColor: '#004a99',
          backgroundColor: 'rgba(0, 74, 153, 0.2)',
          fill: true,
          tension: 0.3,
          pointRadius: 5,
          pointHoverRadius: 7,
        }]
      },
      options: {
        scales: {
          y: {
            reverse: true, // passo migliore è più basso
            ticks: {
              callback: function(value) {
                const min = Math.floor(value / 60);
                const sec = Math.round(value % 60);
                return min + ':' + (sec < 10 ? '0' + sec : sec);
              }
            },
            title: {
              display: true,
              text: 'Minuti al km'
            }
          },
          x: {
            title: {
              display: true,
              text: 'Data'
            }
          }
        },
        plugins: {
          tooltip: {
            callbacks: {
              label: ctx => 'Passo: ' + paceLabels[ctx.dataIndex] + ' min/km'
            }
          }
        }
      }
    });
  });

  // Toggle schermate
  document.getElementById('showListBtn').addEventListener('click', () => {
    renderWorkoutList();
    document.getElementById('mainScreen').style.display = 'none';
    document.getElementById('allWorkoutsScreen').style.display = 'block';
  });

  document.getElementById('backBtn').addEventListener('click', () => {
    document.getElementById('allWorkoutsScreen').style.display = 'none';
    document.getElementById('mainScreen').style.display = 'block';
  });

</script>

</body>
</html>
