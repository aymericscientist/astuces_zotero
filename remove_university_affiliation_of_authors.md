1) Selectionner votre bibliothèque puis sélectionner tous les articles de la bibliothèque (ctrl + a) dont vous souhaitez renommer le pdf
2) Cliquer sur outil --> développeur --> Run JavaScript
![image](https://github.com/user-attachments/assets/b42d3919-ae3a-4b99-bdf7-3b529fbabeb2)

3) copier tout le code suivant en dessous de code : 

///// COMMENCER A COPIER APRES CETTE LIGNE /////

<pre><code>

// Zotero 8 — Exact equivalent of
// "Rename file to match parent item" + set attachment title to "PDF"

const zp = Zotero.getActiveZoteroPane();
if (!zp) {
  Zotero.alert(null, "Renommer (parent)", "Fenêtre Zotero introuvable.");
  return;
}

const selected = zp.getSelectedItems() || [];
if (!selected.length) {
  Zotero.alert(null, "Renommer (parent)", "Aucun élément sélectionné.");
  return;
}

// Collect attachment IDs (attachments directly + children of parent items)
const attachmentIDs = new Set();
for (const it of selected) {
  try {
    if (it.isAttachment && it.isAttachment()) {
      attachmentIDs.add(it.id);
    } else if (it.getAttachments) {
      for (const id of it.getAttachments()) attachmentIDs.add(id);
    }
  } catch (e) {
    Zotero.logError(e);
  }
}

let renamed = 0, retitled = 0, skipped = 0, errors = 0;

for (const id of attachmentIDs) {
  try {
    const att = await Zotero.Items.getAsync(id);
    if (!att || !(att.isFileAttachment && att.isFileAttachment())) { skipped++; continue; }
    if (!att.parentItemID) { skipped++; continue; }

    if (!Zotero.Attachments.isRenameAllowedForType(att.attachmentContentType)) { skipped++; continue; }
    if (att.attachmentLinkMode === Zotero.Attachments.LINK_MODE_LINKED_URL) { skipped++; continue; }

    const parent = await Zotero.Items.getAsync(att.parentItemID);
    if (!parent) { skipped++; continue; }

    const path = await att.getFilePathAsync();
    if (!path) { skipped++; continue; }

    const currentName = path.split(/(\\|\/)/g).pop();
    const ext = currentName.includes(".") ? currentName.split(".").pop() : "";
    const base = Zotero.Attachments.getFileBaseNameFromItem(parent);
    const targetName = ext ? `${base}.${ext}` : base;

    // 1️⃣ Rename file if needed
    if (currentName !== targetName) {
      await att.renameAttachmentFile(targetName);
      renamed++;
    }

    // 2️⃣ Force attachment title to "PDF"
    if (att.getField("title") !== "PDF") {
      att.setField("title", "PDF");
      await att.saveTx();
      retitled++;
    }

  } catch (e) {
    Zotero.logError(e);
    errors++;
  }
}

Zotero.alert(
  null,
  "Renommer (parent)",
  `Terminé.
Fichiers renommés : ${renamed}
Titres remis à "PDF" : ${retitled}
Ignorés : ${skipped}
Erreurs : ${errors}`
);


</code></pre>
///// ARRETER DE COPIER AVANT CETTE LIGNE /////

4) cliquez sur Exécuter

