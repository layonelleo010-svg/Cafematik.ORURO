<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <link rel="manifest" href="manifest.json">
  <meta name="theme-color" content="#4E342E">
  <title>Café Control</title>
  <style>
    /* ----- Estilos generales ----- */
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: 'Segoe UI', system-ui, sans-serif;
      background: #F5F0EB;
      color: #3E2723;
      padding-bottom: 70px;
    }
    .header {
      background: #4E342E;
      color: #FFF8F0;
      padding: 15px;
      text-align: center;
      font-size: 1.3rem;
      font-weight: bold;
      box-shadow: 0 2px 8px rgba(0,0,0,0.2);
    }
    .nav {
      position: fixed; bottom: 0; width: 100%;
      display: flex; justify-content: space-around;
      background: #3E2723; padding: 8px 0;
      box-shadow: 0 -2px 8px rgba(0,0,0,0.2);
      z-index: 100;
    }
    .nav button {
      background: none; border: none; color: #D7CCC8;
      font-size: 0.8rem; display: flex; flex-direction: column; align-items: center;
      padding: 5px 10px; border-radius: 12px; transition: 0.2s;
    }
    .nav button.active { color: #FFD54F; background: rgba(255,255,255,0.1); }
    .nav button svg { width: 24px; height: 24px; margin-bottom: 2px; }
    .container { padding: 20px; max-width: 600px; margin: 0 auto; }
    .card {
      background: #FFFFFF; border-radius: 16px; padding: 16px;
      margin-bottom: 16px; box-shadow: 0 2px 12px rgba(0,0,0,0.06);
    }
    .card h2 { font-size: 1.1rem; margin-bottom: 8px; color: #4E342E; }
    .grid2 { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
    .metric {
      background: #FFF8F0; border-radius: 12px; padding: 12px; text-align: center;
    }
    .metric .value { font-size: 1.5rem; font-weight: bold; color: #4E342E; }
    .metric .label { font-size: 0.8rem; color: #6D4C41; }
    .btn {
      background: #6D4C41; color: white; border: none; border-radius: 25px;
      padding: 10px 20px; font-size: 0.9rem; cursor: pointer; margin: 5px 2px;
    }
    .btn.secondary { background: #A1887F; }
    input, select {
      width: 100%; padding: 10px; margin: 5px 0 12px;
      border: 1px solid #D7CCC8; border-radius: 10px; font-size: 0.95rem;
      background: #FFF8F0;
    }
    table { width: 100%; border-collapse: collapse; font-size: 0.85rem; }
    th { background: #4E342E; color: white; padding: 8px; }
    td { padding: 8px; border-bottom: 1px solid #EFEBE9; }
    .alert { background: #FFF3E0; border-left: 4px solid #FF8F00; padding: 10px; margin: 10px 0; }
    .hidden { display: none; }
    .section { display: none; }
    .section.active { display: block; }
  </style>
</head>
<body>
  <div class="header">☕ Café Control</div>

  <div id="dashboard" class="section active">
    <div class="container">
      <div class="card">
        <h2>📊 Resumen Hoy</h2>
        <div class="grid2">
          <div class="metric"><div class="value" id="hoyVentas">--</div><div class="label">Ventas Bs</div></div>
          <div class="metric"><div class="value" id="hoyGanancia">--</div><div class="label">Ganancia Bs</div></div>
          <div class="metric"><div class="value" id="hoyMargen">--</div><div class="label">Margen %</div></div>
          <div class="metric"><div class="value" id="hoyVasos">--</div><div class="label">Vasos totales</div></div>
        </div>
      </div>
      <div class="card">
        <h2>⚖️ Punto de Equilibrio</h2>
        <div class="metric" style="background:#EFEBE9"><span class="value" id="pEquilibrio">--</span><br><span class="label">vasos necesarios/mes</span></div>
        <div id="barraProgreso" style="height:10px; background:#D7CCC8; border-radius:5px; margin-top:8px;">
          <div id="progresoFill" style="height:100%; width:0%; background:#4CAF50; border-radius:5px;"></div>
        </div>
      </div>
    </div>
  </div>

  <div id="ventas" class="section">
    <div class="container">
      <div class="card">
        <h2>📝 Registrar Ventas</h2>
        <label>Fecha</label><input type="date" id="fechaVenta">
        <div id="productosVenta"></div>
        <button class="btn" onclick="registrarVenta()">💾 Guardar Venta</button>
      </div>
      <div class="card">
        <h2>📋 Historial de Ventas</h2>
        <div id="historialVentas"></div>
      </div>
    </div>
  </div>

  <div id="inventario" class="section">
    <div class="container">
      <div class="card">
        <h2>🧾 Insumos</h2>
        <div id="listaInsumos"></div>
        <button class="btn secondary" onclick="agregarInsumo()">+ Nuevo insumo</button>
      </div>
    </div>
  </div>

  <div id="gastos" class="section">
    <div class="container">
      <div class="card">
        <h2>🧾 Gastos Operativos</h2>
        <label>Descripción</label><input id="descGasto" placeholder="Alquiler, pasajes...">
        <label>Monto (Bs)</label><input type="number" id="montoGasto" step="0.01">
        <label>Fecha</label><input type="date" id="fechaGasto">
        <button class="btn" onclick="registrarGasto()">➕ Añadir</button>
        <div id="listaGastos" style="margin-top:15px;"></div>
      </div>
    </div>
  </div>

  <div class="nav">
    <button onclick="showSection('dashboard')" class="active" id="btnDash">
      <svg viewBox="0 0 24 24" fill="currentColor"><path d="M3,13H11V3H3V13M13,21H21V11H13V21M13,3V9H21V3H13M3,21H11V15H3V21Z"/></svg> Inicio
    </button>
    <button onclick="showSection('ventas')" id="btnVentas">
      <svg viewBox="0 0 24 24" fill="currentColor"><path d="M20,2H4C2.9,2,2,2.9,2,4V22L6,18H20C21.1,18,22,17.1,22,16V4C22,2.9,21.1,2,20,2M20,16H5.2L4,17.2V4H20V16Z"/></svg> Ventas
    </button>
    <button onclick="showSection('inventario')" id="btnInventario">
      <svg viewBox="0 0 24 24" fill="currentColor"><path d="M3,4H21V8H19V20H5V8H3V4M8,8H16V10H8V8M8,12H16V14H8V12M8,16H13V18H8V16"/></svg> Insumos
    </button>
    <button onclick="showSection('gastos')" id="btnGastos">
      <svg viewBox="0 0 24 24" fill="currentColor"><path d="M12,2A10,10 0 0,1 22,12A10,10 0 0,1 12,22A10,10 0 0,1 2,12A10,10 0 0,1 12,2M12,4A8,8 0 0,0 4,12A8,8 0 0,0 12,20A8,8 0 0,0 20,12A8,8 0 0,0 12,4M11,17V16H9V14H13V13H10A2,2 0 0,1 8,11V9A2,2 0 0,1 10,7H11V6H13V7H15V9H11V10H14A2,2 0 0,1 16,12V14A2,2 0 0,1 14,16H13V17H11Z"/></svg> Gastos
    </button>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/idb@7/build/umd.js"></script>
  <script>
    // ----- Lógica de la aplicación (app.js) -----
    // (El código completo de la app está en la siguiente sección)
  </script>
</body>
</html>