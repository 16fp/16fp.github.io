<!DOCTYPE html>
<html lang="es">
<head>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calificador 16PF - Estilo Original</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f0f2f5; padding: 20px; color: #333; }
        .container { max-width: 900px; margin: 0 auto; background: white; padding: 30px; border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
        h1, h2 { color: #2c3e50; text-align: center; border-bottom: 2px solid #eee; padding-bottom: 10px; }
        
        .header-controls { display: flex; justify-content: space-between; margin-bottom: 20px; background: #e8f4f8; padding: 15px; border-radius: 8px; }
        select, button { padding: 8px 15px; border-radius: 5px; border: 1px solid #ccc; font-size: 16px; }
        button { background-color: #007bff; color: white; border: none; cursor: pointer; font-weight: bold; transition: 0.2s; }
        button:hover { background-color: #0056b3; }
        .btn-limpiar { background-color: #dc3545; }
        .btn-limpiar:hover { background-color: #a71d2a; }

        /* Estilo para agrupar de 5 en 5 */
        .hoja-respuestas { display: flex; flex-wrap: wrap; gap: 15px; justify-content: center; }
        .bloque-5 { background: #fdfdfd; border: 2px solid #ddd; padding: 10px; border-radius: 6px; width: 80px; text-align: center; }
        .pregunta-row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 5px; }
        .pregunta-row label { font-weight: bold; font-size: 14px; color: #555; width: 30px; text-align: right; margin-right: 5px; }
        .pregunta-row input { width: 35px; padding: 4px; text-align: center; text-transform: uppercase; border: 1px solid #aaa; border-radius: 3px; font-weight: bold; }
        .pregunta-row input:focus { outline: 2px solid #007bff; }

        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { border: 1px solid #ddd; padding: 10px; text-align: center; }
        th { background-color: #2c3e50; color: white; }
        
    </style>
</head>
<body>

<div class="container">
    <h1>Calificador 16PF (Agrupado por 5)</h1>
    
    <div class="header-controls">
        <div>
            <label for="sexo"><strong>Sexo:</strong></label>
            <select id="sexo">
                <option value="M">Masculino (Cuadro 7.d)</option>
                <option value="F">Femenino (Cuadro 7.e)</option>
            </select>
        </div>
        <div>
            <button class="btn-limpiar" onclick="limpiar()">Borrar Todo</button>
            <button onclick="calcular()">Calcular Resultados</button>
        </div>
    </div>

    <h2>Hoja de Respuestas</h2>
    <div id="hoja-respuestas" class="hoja-respuestas">
        </div>

    <div id="resultados" style="display:none; margin-top: 30px;">
        <h2>Perfil de Resultados</h2>
        
        <table>
            <thead>
                <tr>
                    <th>Factor</th>
                    <th>Suma Bruta</th>
                    <th>Estén (1-10)</th>
                </tr>
            </thead>
            <tbody id="tabla-resultados"></tbody>
        </table>

        <div style="margin-top: 40px; background: #fff; padding: 20px; border: 1px solid #ddd; border-radius: 8px;">
            <h3 style="text-align: center; color: #2c3e50;">Gráfica de Perfil 16PF</h3>
            <canvas id="graficaPerfil" height="150"></canvas>
        </div>
    </div>



<script>
    // 1. GENERAR LA HOJA DE RESPUESTAS (De 5 en 5)
    const contenedor = document.getElementById('hoja-respuestas');
    let html = '';
    for (let i = 1; i <= 187; i++) {
        if (i % 5 === 1) html += '<div class="bloque-5">';
        html += `
            <div class="pregunta-row">
                <label>${i}.</label>
                <input type="text" id="q${i}" maxlength="1" oninput="this.value = this.value.toUpperCase().replace(/[^ABC]/g, '')">
            </div>
        `;
        if (i % 5 === 0 || i === 187) html += '</div>';
    }
    contenedor.innerHTML = html;

    // 2. MATRIZ DE CLAVES (PUNTUACIÓN)
    // Extraído de tu archivo "Escalas 16 FP"
    const claves = {
        // --- FACTOR A ---
       // --- FACTOR A ---
        3:   { factor: 'A', A: 2, B: 1, C: 0 },
        26:  { factor: 'A', A: 0, B: 1, C: 2 },
        27:  { factor: 'A', A: 0, B: 1, C: 2 },
        51:  { factor: 'A', A: 0, B: 1, C: 2 },
        52:  { factor: 'A', A: 2, B: 1, C: 0 },
        76:  { factor: 'A', A: 0, B: 1, C: 2 },
        101: { factor: 'A', A: 2, B: 1, C: 0 },
        126: { factor: 'A', A: 2, B: 1, C: 0 },
        151: { factor: 'A', A: 0, B: 1, C: 2 },
        176: { factor: 'A', A: 2, B: 1, C: 0 },

        // --- FACTOR B (Atención: Solo hay 1 correcta) ---
        28:  { factor: 'B', A: 0, B: 1, C: 0 },
        53:  { factor: 'B', A: 0, B: 1, C: 0 },
        54:  { factor: 'B', A: 0, B: 1, C: 0 },
        77:  { factor: 'B', A: 0, B: 0, C: 1 },
        78:  { factor: 'B', A: 0, B: 1, C: 0 },
        102: { factor: 'B', A: 0, B: 0, C: 1 },
        103: { factor: 'B', A: 0, B: 1, C: 0 },
        127: { factor: 'B', A: 0, B: 0, C: 1 },
        128: { factor: 'B', A: 0, B: 1, C: 0 },
        152: { factor: 'B', A: 1, B: 0, C: 0 },
        153: { factor: 'B', A: 0, B: 0, C: 1 },
        177: { factor: 'B', A: 1, B: 0, C: 0 },
        178: { factor: 'B', A: 1, B: 0, C: 0 },

        // --- FACTOR C --- 
        4:   { factor: 'C', A: 2, B: 1, C: 0 },
        5:   { factor: 'C', A: 0, B: 1, C: 2 },
        29:  { factor: 'C', A: 0, B: 1, C: 2 },
        30:  { factor: 'C', A: 2, B: 1, C: 0 },
        55:  { factor: 'C', A: 2, B: 1, C: 0 },
        79:  { factor: 'C', A: 0, B: 1, C: 2 },
        80:  { factor: 'C', A: 0, B: 1, C: 2 },
        104: { factor: 'C', A: 2, B: 1, C: 0 },
        105: { factor: 'C', A: 2, B: 1, C: 0 },
        129: { factor: 'C', A: 0, B: 1, C: 2 },
        130: { factor: 'C', A: 2, B: 1, C: 0 },
        154: { factor: 'C', A: 0, B: 1, C: 2 },
        179: { factor: 'C', A: 2, B: 1, C: 0 },

        // --- FACTOR E ---
        6:   { factor: 'E', A: 0, B: 1, C: 2 },
        7:   { factor: 'E', A: 2, B: 1, C: 0 },
        31:  { factor: 'E', A: 0, B: 1, C: 2 },
        32:  { factor: 'E', A: 0, B: 1, C: 2 },
        56:  { factor: 'E', A: 2, B: 1, C: 0 },
        57:  { factor: 'E', A: 0, B: 1, C: 2 },
        81:  { factor: 'E', A: 0, B: 1, C: 2 },
        106: { factor: 'E', A: 0, B: 1, C: 2 },
        131: { factor: 'E', A: 2, B: 1, C: 0 },
        155: { factor: 'E', A: 2, B: 1, C: 0 },
        156: { factor: 'E', A: 2, B: 1, C: 0 },
        180: { factor: 'E', A: 2, B: 1, C: 0 },
        181: { factor: 'E', A: 2, B: 1, C: 0 },

        // --- FACTOR F ---
        8:   { factor: 'F', A: 0, B: 1, C: 2 },
        33:  { factor: 'F', A: 2, B: 1, C: 0 },
        58:  { factor: 'F', A: 2, B: 1, C: 0 },
        82:  { factor: 'F', A: 0, B: 1, C: 2 },
        83:  { factor: 'F', A: 2, B: 1, C: 0 },
        107: { factor: 'F', A: 0, B: 1, C: 2 },
        108: { factor: 'F', A: 0, B: 1, C: 2 },
        132: { factor: 'F', A: 2, B: 1, C: 0 },
        133: { factor: 'F', A: 2, B: 1, C: 0 },
        157: { factor: 'F', A: 0, B: 1, C: 2 },
        158: { factor: 'F', A: 0, B: 1, C: 2 },
        182: { factor: 'F', A: 2, B: 1, C: 0 },
        183: { factor: 'F', A: 2, B: 1, C: 0 },

        // --- FACTOR G ---
        9:   { factor: 'G', A: 0, B: 1, C: 2 },
        34:  { factor: 'G', A: 0, B: 1, C: 2 },
        59:  { factor: 'G', A: 0, B: 1, C: 2 },
        84:  { factor: 'G', A: 0, B: 1, C: 2 },
        109: { factor: 'G', A: 2, B: 1, C: 0 },
        134: { factor: 'G', A: 2, B: 1, C: 0 },
        159: { factor: 'G', A: 0, B: 1, C: 2 },
        160: { factor: 'G', A: 2, B: 1, C: 0 },
        184: { factor: 'G', A: 2, B: 1, C: 0 },
        185: { factor: 'G', A: 2, B: 1, C: 0 },

        // --- FACTOR H ---
        10:  { factor: 'H', A: 2, B: 1, C: 0 },
        35:  { factor: 'H', A: 0, B: 1, C: 2 },
        36:  { factor: 'H', A: 2, B: 1, C: 0 },
        60:  { factor: 'H', A: 0, B: 1, C: 2 },
        61:  { factor: 'H', A: 0, B: 1, C: 2 },
        85:  { factor: 'H', A: 0, B: 1, C: 2 },
        86:  { factor: 'H', A: 0, B: 1, C: 2 },
        110: { factor: 'H', A: 2, B: 1, C: 0 },
        111: { factor: 'H', A: 2, B: 1, C: 0 },
        135: { factor: 'H', A: 2, B: 1, C: 0 },
        136: { factor: 'H', A: 2, B: 1, C: 0 },
        161: { factor: 'H', A: 0, B: 1, C: 2 },
        186: { factor: 'H', A: 2, B: 1, C: 0 },
        
                // --- FACTOR I ---
        11:  { factor: 'I', A: 0, B: 1, C: 2 },
        12:  { factor: 'I', A: 2, B: 1, C: 0 },
        37:  { factor: 'I', A: 2, B: 1, C: 0 },
        62:  { factor: 'I', A: 0, B: 1, C: 2 },
        87:  { factor: 'I', A: 0, B: 1, C: 2 },
        112: { factor: 'I', A: 2, B: 1, C: 0 },
        137: { factor: 'I', A: 0, B: 1, C: 2 },
        138: { factor: 'I', A: 2, B: 1, C: 0 },
        162: { factor: 'I', A: 0, B: 1, C: 2 },
        163: { factor: 'I', A: 2, B: 1, C: 0 },

        // --- FACTOR L ---
        13:  { factor: 'L', A: 0, B: 1, C: 2 },
        38:  { factor: 'L', A: 2, B: 1, C: 0 },
        63:  { factor: 'L', A: 0, B: 1, C: 2 },
        64:  { factor: 'L', A: 0, B: 1, C: 2 },
        88:  { factor: 'L', A: 2, B: 1, C: 0 },
        89:  { factor: 'L', A: 0, B: 1, C: 2 },
        113: { factor: 'L', A: 2, B: 1, C: 0 },
        114: { factor: 'L', A: 2, B: 1, C: 0 },
        139: { factor: 'L', A: 0, B: 1, C: 2 },
        164: { factor: 'L', A: 2, B: 1, C: 0 },

        // --- FACTOR M ---
        14:  { factor: 'M', A: 0, B: 1, C: 2 },
        15:  { factor: 'M', A: 0, B: 1, C: 2 },
        39:  { factor: 'M', A: 2, B: 1, C: 0 },
        40:  { factor: 'M', A: 2, B: 1, C: 0 },
        65:  { factor: 'M', A: 2, B: 1, C: 0 },
        90:  { factor: 'M', A: 0, B: 1, C: 2 },
        91:  { factor: 'M', A: 2, B: 1, C: 0 },
        115: { factor: 'M', A: 2, B: 1, C: 0 },
        116: { factor: 'M', A: 2, B: 1, C: 0 },
        140: { factor: 'M', A: 2, B: 1, C: 0 },
        141: { factor: 'M', A: 0, B: 1, C: 2 },
        165: { factor: 'M', A: 0, B: 1, C: 2 },
        166: { factor: 'M', A: 0, B: 1, C: 2 },

        // --- FACTOR N ---
        16:  { factor: 'N', A: 0, B: 1, C: 2 },
        17:  { factor: 'N', A: 2, B: 1, C: 0 },
        41:  { factor: 'N', A: 0, B: 1, C: 2 },
        42:  { factor: 'N', A: 2, B: 1, C: 0 },
        66:  { factor: 'N', A: 0, B: 1, C: 2 },
        67:  { factor: 'N', A: 0, B: 1, C: 2 },
        92:  { factor: 'N', A: 0, B: 1, C: 2 },
        117: { factor: 'N', A: 2, B: 1, C: 0 },
        142: { factor: 'N', A: 2, B: 1, C: 0 },
        167: { factor: 'N', A: 2, B: 1, C: 0 },

        // --- FACTOR O ---
        18:  { factor: 'O', A: 2, B: 1, C: 0 },
        19:  { factor: 'O', A: 0, B: 1, C: 2 },
        43:  { factor: 'O', A: 2, B: 1, C: 0 },
        44:  { factor: 'O', A: 0, B: 1, C: 2 },
        68:  { factor: 'O', A: 0, B: 1, C: 2 },
        69:  { factor: 'O', A: 2, B: 1, C: 0 },
        93:  { factor: 'O', A: 0, B: 1, C: 2 },
        94:  { factor: 'O', A: 2, B: 1, C: 0 },
        118: { factor: 'O', A: 2, B: 1, C: 0 },
        119: { factor: 'O', A: 2, B: 1, C: 0 },
        143: { factor: 'O', A: 2, B: 1, C: 0 },
        144: { factor: 'O', A: 0, B: 1, C: 2 },
        168: { factor: 'O', A: 0, B: 1, C: 2 },

        // --- FACTOR Q1 ---
        20:  { factor: 'Q1', A: 2, B: 1, C: 0 },
        21:  { factor: 'Q1', A: 0, B: 1, C: 2 },
        45:  { factor: 'Q1', A: 0, B: 1, C: 2 },
        46:  { factor: 'Q1', A: 2, B: 1, C: 0 },
        70:  { factor: 'Q1', A: 2, B: 1, C: 0 },
        95:  { factor: 'Q1', A: 0, B: 1, C: 2 },
        120: { factor: 'Q1', A: 0, B: 1, C: 2 },
        145: { factor: 'Q1', A: 2, B: 1, C: 0 },
        169: { factor: 'Q1', A: 2, B: 1, C: 0 },
        170: { factor: 'Q1', A: 0, B: 1, C: 2 },

        // --- FACTOR Q2 ---
        22:  { factor: 'Q2', A: 0, B: 1, C: 2 },
        47:  { factor: 'Q2', A: 2, B: 1, C: 0 },
        71:  { factor: 'Q2', A: 2, B: 1, C: 0 },
        72:  { factor: 'Q2', A: 2, B: 1, C: 0 },
        96:  { factor: 'Q2', A: 0, B: 1, C: 2 },
        97:  { factor: 'Q2', A: 0, B: 1, C: 2 },
        121: { factor: 'Q2', A: 0, B: 1, C: 2 },
        122: { factor: 'Q2', A: 0, B: 1, C: 2 },
        146: { factor: 'Q2', A: 2, B: 1, C: 0 },
        171: { factor: 'Q2', A: 2, B: 1, C: 0 },

         // --- FACTOR Q3 ---
        23:  { factor: 'Q3', A: 0, B: 1, C: 2 },
        24:  { factor: 'Q3', A: 0, B: 1, C: 2 },
        48:  { factor: 'Q3', A: 2, B: 1, C: 0 },
        73:  { factor: 'Q3', A: 2, B: 1, C: 0 },
        98:  { factor: 'Q3', A: 2, B: 1, C: 0 },
        123:  { factor: 'Q3', A: 0, B: 1, C: 2 },
        147:  { factor: 'Q3', A: 0, B: 1, C: 2 },
        148:  { factor: 'Q3', A: 2, B: 1, C: 0 },
        172:  { factor: 'Q3', A: 0, B: 1, C: 2 },
        173:  { factor: 'Q3', A: 2, B: 1, C: 0 },


         // --- FACTOR Q4 ---
        25:  { factor: 'Q4', A: 0, B: 1, C: 2 },
        49:  { factor: 'Q4', A: 2, B: 1, C: 0 },
        50:  { factor: 'Q4', A: 2, B: 1, C: 0 },
        74:  { factor: 'Q4', A: 2, B: 1, C: 0 },
        75:  { factor: 'Q4', A: 0, B: 1, C: 2 },
        99:  { factor: 'Q4', A: 2, B: 1, C: 0 },
        100:  { factor: 'Q4', A: 0, B: 1, C: 2 },
        124:  { factor: 'Q4', A: 2, B: 1, C: 0 },
        125:  { factor: 'Q4', A: 0, B: 1, C: 2 },
        149:  { factor: 'Q4', A: 2, B: 1, C: 0 },
        150:  { factor: 'Q4', A: 0, B: 1, C: 2 },
        174:  { factor: 'Q4', A: 2, B: 1, C: 0 },
        175:  { factor: 'Q4', A: 0, B: 1, C: 2 },





    };

    // 3. TABLAS DE BAREMOS (Normas Mexicanas - Límites superiores de cada Estén)
    // Basado en tu PDF Cuadro 7.d y 7.e
    const baremos = {
        M: {
            // Factor A: Estén 1(0-3), 2(4-5), 3(6-7), 4(8), 5(9-10), 6(11), 7(12-13), 8(14-15), 9(16), 10(17-20)
            A: [3, 5, 7, 8, 10, 12, 13, 15, 16, 20],
            B: [1, 2, 4, 5, 6, 7, 8, 8, 9, 13],
            C: [9, 12, 14, 17, 19, 21, 23, 24, 25, 26],
            E: [5, 6, 7, 9, 11, 13, 15, 18, 20, 26],
            F: [4, 6, 8, 10, 13, 15, 17, 20, 22, 2],
            G: [8, 9, 11, 12, 14, 16, 17, 18, 19, 20],
            H: [6, 7, 10, 13, 17, 19, 22, 24, 25, 26],
            I: [1, 3, 4, 5, 7, 9, 10, 12, 13, 20],
            L: [2, 3, 5, 7, 9, 10, 12, 14, 15, 20],
            M: [6, 7, 9, 10, 12, 14, 16, 17, 18, 26],
            N: [5, 6, 8, 10, 11, 12, 14, 15, 17, 20],
            O: [3, 5, 6, 7, 10, 11, 14, 16, 19, 26],
            Q1: [4, 6, 7, 9, 10, 12, 14, 15, 16, 20],
            Q2: [3, 5, 7, 8, 10, 12, 13, 15, 17, 20],
            Q3: [5, 8, 10, 12, 13, 15, 16, 17, 18, 20],
            Q4: [1, 2, 3, 5, 7, 9, 11, 13, 16, 26],
        },
        F: {
            // Ejemplo hipotético de mujeres para Factor A (Revisar tu Cuadro 7.e)
            A: [4, 6, 8, 9, 11, 12, 14, 16, 18, 20],
            B: [2, 3, 4, 5, 6, 7, 8, 9, 10, 13],
            C: [12, 13, 14, 17, 19, 20, 22, 23, 25, 26],
            E: [4, 5, 6, 8, 10, 12, 14, 16, 17, 26],
            F: [3, 6, 8, 10, 12, 14, 16, 17, 21, 26],
            G: [7, 8, 9, 11, 13, 15, 16, 17, 18, 20],
            H: [5, 7, 9, 12, 14, 16, 20, 23, 24, 26],
            I: [3, 5, 6, 7, 10, 12, 14, 15, 16, 20],
            L: [4, 5, 6, 7, 8, 10, 11, 13, 14, 20],
            M: [6, 8, 9, 10, 13, 15, 16, 18, 20, 26],
            N: [5, 7, 8, 9, 10, 12, 13, 14, 15, 20],
            O: [2, 3, 5, 8, 10, 11, 13, 14, 16, 26],
            Q1: [3, 5, 7, 9, 11, 12, 14, 15, 16, 20],
            Q2: [5, 6, 8, 9, 11, 12, 14, 16, 17, 20],
            Q3: [4, 6, 9, 11, 12, 15, 16, 17, 18, 20],
            Q4: [2, 3, 4, 6, 8, 10, 12, 15, 17, 26],
        }
    };

    // 4. LÓGICA DE CALIFICACIÓN
    function calcular() {
        // Inicializar contadores en 0
        const puntajesBrutos = { A:0, B:0, C:0, E:0, F:0, G:0, H:0, I:0, L:0, M:0, N:0, O:0, Q1:0, Q2:0, Q3:0, Q4:0 };
        const sexo = document.getElementById('sexo').value;

        // Sumar puntos
        for (let i = 1; i <= 187; i++) {
            let respuesta = document.getElementById('q' + i).value;
            if (respuesta && claves[i]) {
                let factor = claves[i].factor;
                if (claves[i][respuesta] !== undefined) {
                    puntajesBrutos[factor] += claves[i][respuesta];
                }
            }
        }

        mostrarResultados(puntajesBrutos, sexo);
    }

    // Calcular el Estén cruzando la suma bruta con el array de baremos
    function obtenerEsten(sexo, factor, bruta) {
        if (!baremos[sexo] || !baremos[sexo][factor]) return "Falta Baremo";
        
        let limites = baremos[sexo][factor];
        for (let i = 0; i < limites.length; i++) {
            if (bruta <= limites[i]) {
                return i + 1; // El índice 0 es el Estén 1, el índice 1 es el Estén 2, etc.
            }
        }
        return 10; // Si supera el máximo, es 10.
    }

   function mostrarResultados(brutas, sexo) {
        document.getElementById('resultados').style.display = 'block';
        let tbody = document.getElementById('tabla-resultados');
        tbody.innerHTML = ''; 

        // Arrays para guardar los datos de la gráfica
        let factoresArr = [];
        let estenesArr = [];

        for (const [factor, bruta] of Object.entries(brutas)) {
            let esten = obtenerEsten(sexo, factor, bruta);
            
            // Guardamos los datos para la gráfica
            factoresArr.push(factor);
            estenesArr.push(esten);
            
            tbody.innerHTML += `
                <tr>
                    <td><strong>${factor}</strong></td>
                    <td>${bruta}</td>
                    <td><strong>${esten}</strong></td>
                </tr>
            `;
        }

        // Llamamos a la función que dibuja la gráfica con los arrays llenos
        dibujarGrafica(factoresArr, estenesArr);
    }

    function limpiar() {
        for (let i = 1; i <= 187; i++) {
            document.getElementById('q' + i).value = '';
        }
        document.getElementById('resultados').style.display = 'none';
        window.scrollTo(0, 0);
    }

    // 5. CONFIGURACIÓN DE LA GRÁFICA (Chart.js)
    let chartInstancia = null; // Variable global para destruir la gráfica previa si se recalcula

    function dibujarGrafica(factores, estenes) {
        const ctx = document.getElementById('graficaPerfil').getContext('2d');
        
        // Si ya existe una gráfica, la destruimos para no encimar datos
        if (chartInstancia) {
            chartInstancia.destroy(); 
        }

        chartInstancia = new Chart(ctx, {
            type: 'line', // Tipo línea
            data: {
                labels: factores, // El eje Y (A, B, C...)
                datasets: [{
                    label: 'Estén',
                    data: estenes, // Los puntos del 1 al 10
                    borderColor: '#2c3e50', // Color de la línea
                    backgroundColor: '#007bff', // Color de los puntos
                    borderWidth: 2,
                    pointRadius: 6, // Tamaño del punto
                    pointHoverRadius: 8,
                    fill: false,
                    tension: 0.1 // Curvatura ligera de la línea
                }]
            },
            options: {
                indexAxis: 'y', // Voltea la gráfica para que los factores queden en vertical
                responsive: true,
                scales: {
                    x: {
                        min: 1,
                        max: 10,
                        ticks: {
                            stepSize: 1, // Para que muestre todos los números del 1 al 10
                            font: { weight: 'bold' }
                        },
                        title: { display: true, text: 'Puntuación Estén (1 - 10)', font: { size: 14, weight: 'bold' } }
                    },
                    y: {
                        title: { display: true, text: 'Factores', font: { size: 14, weight: 'bold' } },
                        ticks: { font: { weight: 'bold', size: 14 } }
                    }
                },
                plugins: {
                    legend: { display: false } // Ocultamos la leyenda porque es redundante
                }
            }
        });
    }

</script>
</body>
</html>
