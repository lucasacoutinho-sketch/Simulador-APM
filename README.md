/**
 * API REST APM 4.0 - Diagnóstico Automático por Condição de Sensor (Novo vs Antigo)
 */

function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    
    // Se a planilha estiver vazia, cria os cabeçalhos das 12 colunas
    if (sheet.getLastRow() === 0) {
      sheet.appendRow([
        "Data/Hora",
        "Ativo",
        "Condição do Sensor",
        "Horímetro Total (h)",
        "Vibração RMS (mm/s)",
        "Temperatura (°C)",
        "Corrente L1 (A)",
        "Energia (kWh)",
        "Índice de Saúde",
        "Estado da Operação",
        "Tipo de Manutenção",
        "Ação Recomendada / Diagnóstico"
      ]);
    }
    
    // Lê os dados JSON enviados pelo simulador
    var data = {};
    if (e && e.postData && e.postData.contents) {
      data = JSON.parse(e.postData.contents);
    } else if (e && e.parameter) {
      data = e.parameter;
    }
    
    var timestamp = new Date();
    var ativo = data.asset || data.ativo || "Skid MB-01";
    
    // Identifica se o sensor é "NOVO" ou "ANTIGO" (padrão: "ANTIGO")
    var condicaoSensor = (data.sensorCondition || data.condicaoSensor || data.sensorAge || "ANTIGO").toString().toUpperCase();
    if (condicaoSensor !== "NOVO" && condicaoSensor !== "ANTIGO") {
      condicaoSensor = "ANTIGO";
    }
    
    var horimetro = data.horimetroTotal || data.horimetro || data.horimetroTrip || "";
    
    // Mapeamento das leituras (Inglês e Português)
    var vibracao = parseFloat(data.vibration || data.vibracaoRMS || data.vibracao || 0);
    var temperatura = parseFloat(data.temperature || data.temperatura || 0);
    var corrente = parseFloat(data.current || data.correnteL1 || data.corrente || 0);
    
    var energia = data.totalKWh || data.energiaAcumulada || data.energia || "";
    var saude = data.healthIndex || data.indiceSaude || data.saude || "100%";
    
    // LÓGICA DE AVALIAÇÃO DE LIMIARES E DIAGNÓSTICO DEDICADO
    var estado = "NORMAL";
    var tipoManutencao = "NENHUMA";
    var acaoRecomendada = "Operação dentro dos parâmetros nominais.";
    
    // 1. CONDIÇÃO CRÍTICA (EMERGÊNCIA / CORRETIVA)
    if (temperatura >= 80 || vibracao >= 7.1 || corrente >= 45) {
      estado = "CRÍTICO";
      tipoManutencao = "CORRETIVA";
      
      var motivo = "";
      if (temperatura >= 80) motivo = "Sobretemperatura PT100 ≥ 80°C";
      else if (vibracao >= 7.1) motivo = "Vibração Acelerômetro ≥ 7.1 mm/s";
      else motivo = "Sobrecarga TC Bipartido ≥ 45A";
      
      if (condicaoSensor === "NOVO") {
        acaoRecomendada = "PARADA IMEDIATA! [" + motivo + "]. SENSOR NOVO EM FALHA: Checar instalação incorreta, curto-circuito/fiação, fixação solta ou defeito de fabricação.";
      } else {
        acaoRecomendada = "PARADA IMEDIATA! [" + motivo + "]. SENSOR ANTIGO EM FALHA CRÍTICA: Fim de vida útil atingido. Substituir (trocar) o sensor/inserto e inspecionar o ativo.";
      }
      
    // 2. CONDIÇÃO DE ALERTA (PREVENTIVA / PREDITIVA)
    } else if (temperatura >= 65 || vibracao >= 4.5 || corrente >= 35) {
      estado = "ALERTA";
      tipoManutencao = "PREVENTIVA / PREDITIVA";
      
      var motivoAlerta = "";
      if (temperatura >= 65) motivoAlerta = "Temperatura elevada PT100 ≥ 65°C";
      else if (vibracao >= 4.5) motivoAlerta = "Vibração elevada Acelerômetro ≥ 4.5 mm/s";
      else motivoAlerta = "Corrente elevada TC Bipartido ≥ 35A";
      
      if (condicaoSensor === "NOVO") {
        acaoRecomendada = "INVESTIGAÇÃO TÉCNICA (Sensor NOVO): [" + motivoAlerta + "]. Verificar mau contato no conector M12, calibração inicial, aperto mecânico ou interferência eletromagnética.";
      } else {
        acaoRecomendada = "AGENDAR PARADA PROGRAMADA (Sensor ANTIGO): [" + motivoAlerta + "]. Realizar manutenção preventiva, inspeção de desgaste e agendar possível troca preventiva.";
      }
    }
    
    // Gravação da linha na planilha
    sheet.appendRow([
      timestamp,
      ativo,
      condicaoSensor,
      horimetro,
      vibracao,
      temperatura,
      corrente,
      energia,
      saude,
      estado,
      tipoManutencao,
      acaoRecomendada
    ]);
    
    return ContentService.createTextOutput(JSON.stringify({ 
      status: "sucesso",
      estado: estado,
      condicaoSensor: condicaoSensor,
      tipoManutencao: tipoManutencao,
      acaoRecomendada: acaoRecomendada
    })).setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({ status: "erro", message: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  return ContentService.createTextOutput("API APM 4.0 - Diagnóstico por Condição de Sensor Ativo.")
    .setMimeType(ContentService.MimeType.TEXT);
}

function doOptions(e) {
  return ContentService.createTextOutput("")
    .setMimeType(ContentService.MimeType.TEXT);
}
