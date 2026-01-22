const SHEET_NAME = 'Datos';

function getSheet_(){
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  let sh = ss.getSheetByName(SHEET_NAME);
  if(!sh){
    sh = ss.insertSheet(SHEET_NAME);
    sh.appendRow(['id','name','date','sex']);
  }
  return sh;
}

function doGet(e){
  const action = (e && e.parameter && e.parameter.action) ? e.parameter.action : 'ping';

  if(action === 'list'){
    const sh = getSheet_();
    const values = sh.getDataRange().getValues();
    const rows = values.slice(1); // sin header

    const items = rows
      .filter(r => r[0] || r[1] || r[2] || r[3])
      .map(r => ({
        id: String(r[0] || ''),
        name: String(r[1] || ''),
        date: String(r[2] || ''),
        sex: String(r[3] || '')
      }));

    return ContentService
      .createTextOutput(JSON.stringify({ ok:true, items }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  return ContentService
    .createTextOutput(JSON.stringify({ ok:true, msg:'Calendar API OK. Usa ?action=list' }))
    .setMimeType(ContentService.MimeType.JSON);
}

function doPost(e){
  if(!e || !e.postData || !e.postData.contents){
    return ContentService
      .createTextOutput(JSON.stringify({ ok:false, error:'No postData. Prueba desde la web, no con Ejecutar.' }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  let body = {};
  try { body = JSON.parse(e.postData.contents); } catch(err){}

  const action = body.action || 'save';
  if(action === 'save'){
    const items = Array.isArray(body.items) ? body.items : [];
    const sh = getSheet_();

    sh.clear();
    sh.appendRow(['id','name','date','sex']);

    if(items.length){
      const values = items.map(x => [
        String(x.id || ''),
        String(x.name || ''),
        String(x.date || ''),
        String(x.sex || '')
      ]);
      sh.getRange(2,1,values.length,4).setValues(values);
    }

    return ContentService
      .createTextOutput(JSON.stringify({ ok:true, saved: items.length }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  return ContentService
    .createTextOutput(JSON.stringify({ ok:false, error:'Acción no válida' }))
    .setMimeType(ContentService.MimeType.JSON);
}
