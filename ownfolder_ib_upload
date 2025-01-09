// Efficient Google Apps Script for renaming and uploading files
function renameDownloadedFilesFinal2(folderId) {
  const folder = DriveApp.getFolderById(folderId);
  const files = folder.getFiles();

  while (files.hasNext()) {
    const file = files.next();
    const fileName = file.getName();

    if (/.*asn_data.*/.test(fileName)) {
      file.setName("Upload_ASN_PBs.csv");
    } else if (/.*box_status_data.*/.test(fileName)) {
      file.setName("Upload_Boxes.csv");
    } else if (/.*wh_asn.*/.test(fileName)) {
      file.setName("Upload_WH_ASN.csv");
    } else if (fileName.includes("pendencies_JED01")) {
      file.setName("Wall.csv");
      SpreadsheetApp.getUi().alert("File Renamed!");
    }
  }

  countDownBeforeExecution(1, 10, () => uploadSpecificFilesToGoogleSheet(folderId));
}

function getFolderIdFromSheet(folderId) {
}

function countDownBeforeExecution(start, end, callback) {
  for (let i = start; i <= end; i++) {
    Logger.log(`Counting: ${i}`);
    Utilities.sleep(1000); // Delay by 1 second
  }
  callback();
}

function uploadSpecificFilesToGoogleSheet() {
  const folderId = getFolderIdFromSheet();
  const spreadsheetId = "1qOkZrhiT-TkOKNcK5kmkK_rvH7uTOvfZVljtKJ0eN0U";
  const fileMappings = [
    { fileName: "Upload_ASN_PBs.csv", sheetName: "Upload_ASN_PBs", timestampCell: "B1" },
    { fileName: "Upload_WH_ASN.csv", sheetName: "Upload_WH_ASN", timestampCell: "B2" },
    { fileName: "Upload_Boxes.csv", sheetName: "Upload_Boxes", timestampCell: "B3" }
  ];

  const folder = DriveApp.getFolderById(folderId);

  fileMappings.forEach(mapping => {
    const files = folder.getFilesByName(mapping.fileName);

    if (files.hasNext()) {
      const file = files.next();
      const fileContent = file.getBlob().getDataAsString();
      const delimiter = fileContent.includes("\t") ? "\t" : ",";
      const csvData = Utilities.parseCsv(fileContent, delimiter);

      if (csvData && csvData.length > 0 && csvData[0].length > 0) {
        const spreadsheet = SpreadsheetApp.openById(spreadsheetId);
        const sheet = spreadsheet.getSheetByName(mapping.sheetName) || spreadsheet.insertSheet(mapping.sheetName);

        if (mapping.sheetName === "Upload_WH_ASN") {
          const lastRow = sheet.getLastRow();
          if (sheet.getLastColumn() >= 20) {
            sheet.getRange(1, 1, lastRow, 19).clear();
          }
        } else {
          sheet.clear();
        }

        sheet.getRange(1, 1, csvData.length, csvData[0].length).setValues(csvData);
        updateTimestamp(spreadsheetId, mapping.timestampCell);
        Logger.log(`File '${mapping.fileName}' uploaded to '${mapping.sheetName}'.`);
      } else {
        Logger.log(`Invalid or empty CSV data in file: ${mapping.fileName}`);
        updateTimestamp(spreadsheetId, mapping.timestampCell, "No file uploaded");
      }
    } else {
      Logger.log(`File not found: ${mapping.fileName}`);
      updateTimestamp(spreadsheetId, mapping.timestampCell, "No file uploaded");
    }
  });

  countDownBeforeExecution(1, 10, deleteOldFiles);
}

function updateTimestamp(spreadsheetId, timestampCell, message) {
  const sheet = SpreadsheetApp.openById(spreadsheetId).getSheetByName("Time") || SpreadsheetApp.openById(spreadsheetId).insertSheet("Time");
  const timestampRange = sheet.getRange(timestampCell);

  timestampRange.setValue(message || new Date());

  const userName = Session.getEffectiveUser().getEmail();
  sheet.getRange("B4").setValue(userName);
}

function deleteOldFiles(folderId) {
  const folderId = getFolderIdFromSheet();
  const folder = DriveApp.getFolderById(folderId);
  const files = folder.getFiles();

  while (files.hasNext()) {
    files.next().setTrashed(true);
  }
}
