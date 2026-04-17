1FR) Selectionner votre bibliothèque puis sélectionner tous les articles de la bibliothèque (ctrl + a)  
1EN) Select your library and then select all the items in the library (ctrl + a)

2FR) Cliquer sur outil --> développeur --> Run JavaScript  
2EN) Click on Tools --> Developer --> Run JavaScript
![image](https://github.com/user-attachments/assets/b42d3919-ae3a-4b99-bdf7-3b529fbabeb2)

3FR) Cliquer sur le petit <img width="54" height="73" alt="image" src="https://github.com/user-attachments/assets/ce0c7549-5f41-4929-82d9-99bbb2e21887" /> en haut gauche de la fenêtre ci-dessous
 :  
3EN) Click on the small <img width="54" height="73" alt="image" src="https://github.com/user-attachments/assets/ce0c7549-5f41-4929-82d9-99bbb2e21887" /> in the top left corner of the window below:  

<pre><code>

(async () => {
    const zoteroPane = Zotero.getActiveZoteroPane();
    const libraryID = zoteroPane.getSelectedLibraryID();
    const selectedCollection = zoteroPane.getSelectedCollection();
    const win = zoteroPane.document.defaultView;

    const Ci = Components.interfaces;
    const Cc = Components.classes;

    function normalizeDOI(raw) {
        if (!raw) return "";
        raw = String(raw).trim();
        raw = raw.replace(/^https?:\/\/(dx\.)?doi\.org\//i, "");
        raw = raw.replace(/^doi:\s*/i, "");
        raw = raw.trim();

        const m = raw.match(/10\.\d{4,9}\/[-._;()/:A-Z0-9]+/i);
        return m ? m[0] : "";
    }

    function getDOIFromExtra(extra) {
        if (!extra) return "";

        let m = extra.match(/(?:^|\n)\s*DOI\s*:\s*(10\.\d{4,9}\/[-._;()/:A-Z0-9]+)\s*(?=$|\n)/i);
        if (m) return normalizeDOI(m[1]);

        m = extra.match(/10\.\d{4,9}\/[-._;()/:A-Z0-9]+/i);
        return m ? normalizeDOI(m[0]) : "";
    }

    function getItemDOI(item) {
        let doi = normalizeDOI(item.getField("DOI"));
        if (doi) return doi;

        const extra = item.getField("extra") || "";
        doi = getDOIFromExtra(extra);
        return doi || "";
    }

    async function getParentItems() {
        if (selectedCollection) {
            return selectedCollection
                .getChildItems()
                .filter(item => item && item.isRegularItem());
        }

        const s = new Zotero.Search();
        s.libraryID = libraryID;

        const ids = await s.search();
        const items = await Zotero.Items.getAsync(ids);

        return items.filter(item => item && item.isRegularItem());
    }

    function hasPDFAttachment(item) {
        const attachmentIDs = item.getAttachments();
        for (const attID of attachmentIDs) {
            const att = Zotero.Items.get(attID);
            if (
                att &&
                att.isAttachment() &&
                att.attachmentContentType === "application/pdf"
            ) {
                return true;
            }
        }
        return false;
    }

    function csvEscape(value) {
        if (value === null || value === undefined) return "";
        const s = String(value).replace(/\r?\n/g, " ").trim();
        return `"${s.replace(/"/g, '""')}"`;
    }

    function getTimestamp() {
        const d = new Date();
        const yyyy = d.getFullYear();
        const mm = String(d.getMonth() + 1).padStart(2, "0");
        const dd = String(d.getDate()).padStart(2, "0");
        const hh = String(d.getHours()).padStart(2, "0");
        const mi = String(d.getMinutes()).padStart(2, "0");
        const ss = String(d.getSeconds()).padStart(2, "0");
        return `${yyyy}-${mm}-${dd}_${hh}-${mi}-${ss}`;
    }

    async function yieldUI(ms = 0) {
        await new Promise(resolve => win.setTimeout(resolve, ms));
    }

    function getDirSvc() {
        return Cc["@mozilla.org/file/directory_service;1"]
            .getService(Ci.nsIProperties);
    }

    function tryGetSpecialDir(key) {
        try {
            return getDirSvc().get(key, Ci.nsIFile);
        }
        catch (e) {
            return null;
        }
    }

    function resolveOutputFile(fileName) {
        const candidates = [
            tryGetSpecialDir("DfltDwnld"),
            tryGetSpecialDir("Desk"),
            tryGetSpecialDir("Home")
        ].filter(Boolean);

        if (!candidates.length) {
            throw new Error("Impossible de resoudre un dossier de sortie");
        }

        for (const dir of candidates) {
            try {
                const file = dir.clone();
                file.append(fileName);
                return file;
            }
            catch (e) {
            }
        }

        throw new Error("Impossible de construire le chemin du fichier de sortie");
    }

    function showMessage(title, message) {
        try {
            Services.prompt.alert(win, title, message);
            return;
        }
        catch (e) {
        }

        try {
            win.alert(message);
        }
        catch (e) {
        }
    }

    let overlayShown = false;

    try {
        Zotero.showZoteroPaneProgressMeter("Analyse des parents sans PDF...", true);
        overlayShown = true;
        Zotero.updateZoteroPaneProgressMeter(0);

        const parentItems = await getParentItems();
        const total = parentItems.length;
        const rows = [];

        if (!total) {
            Zotero.updateZoteroPaneProgressMeter(100);
            await yieldUI(100);

            const msg = "Aucun item parent trouve dans le perimetre courant.";
            if (overlayShown) {
                Zotero.hideZoteroPaneOverlays();
                overlayShown = false;
            }
            showMessage("Export CSV", msg);
            return msg;
        }

        for (let i = 0; i < total; i++) {
            const item = parentItems[i];
            const current = i + 1;
            const percent = Math.floor((current / total) * 92);

            Zotero.updateZoteroPaneProgressMeter(percent);

            if (i % 25 === 0) {
                await yieldUI(0);
            }

            if (hasPDFAttachment(item)) {
                continue;
            }

            const doi = getItemDOI(item);
            if (!doi) {
                continue;
            }

            rows.push({
                itemID: item.id,
                itemKey: item.key,
                itemType: Zotero.ItemTypes.getName(item.itemTypeID) || "",
                year: item.getField("date") || "",
                title: item.getField("title") || "",
                doi: doi,
                doiURL: "https://doi.org/" + doi
            });
        }

        rows.sort((a, b) => {
            const x = (a.doi || "").toLowerCase();
            const y = (b.doi || "").toLowerCase();
            return x.localeCompare(y);
        });

        Zotero.updateZoteroPaneProgressMeter(95);
        await yieldUI(50);

        const fileName = `zotero_parents_without_pdf_doi_${getTimestamp()}.csv`;
        const outputFile = resolveOutputFile(fileName);

        const header = [
            "itemID",
            "itemKey",
            "itemType",
            "year",
            "title",
            "DOI",
            "DOI_URL"
        ];

        const csvLines = [header.join(",")];

        for (const r of rows) {
            csvLines.push([
                csvEscape(r.itemID),
                csvEscape(r.itemKey),
                csvEscape(r.itemType),
                csvEscape(r.year),
                csvEscape(r.title),
                csvEscape(r.doi),
                csvEscape(r.doiURL)
            ].join(","));
        }

        const csvContent = csvLines.join("\n");
        await Zotero.File.putContentsAsync(outputFile.path, csvContent);

        Zotero.updateZoteroPaneProgressMeter(100);
        await yieldUI(150);

        const message = [
            `CSV cree ici :`,
            outputFile.path,
            ``,
            `Nombre de parents sans PDF mais avec DOI : ${rows.length}`,
            `Nombre total de parents analyses : ${total}`
        ].join("\n");

        if (overlayShown) {
            Zotero.hideZoteroPaneOverlays();
            overlayShown = false;
            await yieldUI(100);
        }

        showMessage("Export CSV termine", message);
        return message;
    }
    catch (e) {
        if (overlayShown) {
            Zotero.hideZoteroPaneOverlays();
            overlayShown = false;
        }

        const errorMessage = `Erreur pendant l'export : ${e.message || e}`;
        showMessage("Erreur export CSV", errorMessage);
        throw new Error(errorMessage);
    }
})();

</code></pre>

4FR) cliquez sur Exécuter  
4EN) click on Run  

5FR) rajouter la colonne extra  
5EN) add the extra column

<img width="1818" height="507" alt="Capture d’écran 2026-03-05 202147" src="https://github.com/user-attachments/assets/daeb56c8-8385-4492-b386-5bb9be56894d" />

Attention : il faut répéter cette action à chaque fois que vous voulez mettre à jour le classement FNEGE 2025 concernant vos articles. C'est manuel et non automatique.  
Important: This action must be repeated each time you want to update the FNEGE 2025 ranking for your articles. It is a manual process, not automatic.

Si jamais vous étiez sur la version précédente, voici le script permettant de supprimer tout ce qu'il y a dans le champ "loc dans l'archive" :  
If you were using the previous version, here is the script to delete everything in the "loc in archive" field:

