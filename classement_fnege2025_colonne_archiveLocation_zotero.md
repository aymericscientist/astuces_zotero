1) Selectionner votre bibliothèque puis sélectionner tous les articles de la bibliothèque (ctrl + a)
2) Cliquer sur outil --> développeur --> Run JavaScript
![image](https://github.com/user-attachments/assets/b42d3919-ae3a-4b99-bdf7-3b529fbabeb2)

3) copier tout le code suivant en dessous de code : 

///// COMMENCER A COPIER APRES CETTE LIGNE /////

<pre><code>
// Script Zotero : Ajout / mise a jour du classement FNEGE 2025 dans archiveLocation

function normalize(text) {
  return text
    .replace(/&/g, "et") // remplace les esperluettes par "et"
    .toLowerCase()
    .normalize("NFD").replace(/[\u0300-\u036f]/g, "")
    .replace(/[^a-z0-9]+/g, " ")
    .replace(/\s+/g, " ")
    .trim();
}

// FNEGE 2025 (source: fichier Excel fourni) - 98 revues
const fnegeRanking = {
  "acm transactions on computer human interaction": "1",
  "aslib journal of information management": "3",
  "australasian journal of information systems": "4",
  "big data and society": "3",
  "business and information systems engineering bise": "3",
  "communications of the association for information systems": "3",
  "computers in human behavior": "2",
  "computers and education": "2",
  "computers and security": "2",
  "decision support systems": "2",
  "european journal of information systems": "1",
  "expert systems with applications": "3",
  "government information quarterly": "2",
  "health informatics journal": "4",
  "human computer interaction": "2",
  "ieee software": "3",
  "ieee transactions on knowledge and data engineering": "1",
  "ieee transactions on software engineering": "1",
  "information and management": "1",
  "information communication and society": "2",
  "information processing and management": "3",
  "information society": "3",
  "information systems frontiers": "3",
  "information systems journal": "1",
  "information systems research": "1*",
  "information systems": "2",
  "information technology and people": "3",
  "international journal of electronic commerce": "2",
  "international journal of human computer studies": "2",
  "international journal of information management": "2",
  "international journal of information systems and project management": "4",
  "international journal of medical informatics": "2",
  "internet research": "3",
  "journal of management information systems": "1",
  "journal of strategic information systems": "1",
  "journal of the association for information systems": "1",
  "journal of the american medical informatics association": "2",
  "journal of information technology": "1",
  "journal of information science": "3",
  "journal of computer mediated communication": "2",
  "journal of biomedical informatics": "3",
  "journal of business research": "2",
  "journal of business logistics": "2",
  "journal of the operational research society": "2",
  "journal of enterprise information management": "3",
  "journal of global information management": "3",
  "journal of health informatics": "4",
  "journal of information systems": "3",
  "journal of internet commerce": "4",
  "journal of knowledge management": "2",
  "journal of medical internet research": "3",
  "journal of systems and software": "2",
  "knowledge management research and practice": "3",
  "mis quarterly": "1*",
  "mis quarterly executive": "3",
  "organisation and management": "4",
  "computers industrial engineering": "3",
  "data and knowledge engineering": "2",
  "decision sciences": "2",
  "electronic commerce research": "4",
  "electronic commerce research and applications": "4",
  "electronic markets": "3",
  "enterprise information systems": "3",
  "eurasian business review": "4",
  "information systems and e business management": "4",
  "information technology and management": "4",
  "interacting with computers": "3",
  "international journal of accounting information systems": "4",
  "international journal of electronic business": "4",
  "international journal of enterprise information systems": "4",
  "international journal of information technology and decision making": "4",
  "international journal of it business management": "4",
  "international journal of logistics management": "3",
  "journal of decision systems": "4",
  "journal of electronic commerce research": "4",
  "journal of information systems and technology management": "4",
  "journal of organizational and end user computing": "4",
  "journal of purchasing and supply management": "3",
  "knowledge and process management": "4",
  "online information review": "3",
  "omega the international journal of management science": "2",
  "operations research": "1*",
  "organization science": "1*",
  "production and operations management": "1",
  "research policy": "1*",
  "service industries journal": "4",
  "technological forecasting and social change": "2",
  "technology analysis and strategic management": "3",
  "telecommunications policy": "3",
  "the information society": "3",
  "the journal of strategic information systems": "1",
  "total quality management and business excellence": "4",
  "information and organization": "2",
  "information systems management": "4",
  "it professional": "4",
  "journal of the operational research society jors": "2",
  "systèmes d information et management": "2"
};

const updateFNEGE2025InArchiveLocation = async () => {
  const selectedItems = ZoteroPane.getSelectedItems();
  let count = 0;

  for (let item of selectedItems) {
    if (!item.isRegularItem()) continue;

    const journalRaw = item.getField("publicationTitle") || "";
    const journalNorm = normalize(journalRaw);

    if (!journalNorm || !(journalNorm in fnegeRanking)) continue;

    const ranking = fnegeRanking[journalNorm];

    // Classement pour tri correct dans Zotero (meme logique que ton script)
    let displayRanking;
    if (ranking === "1*") displayRanking = "0.9 (1*)";
    else if (ranking === "1") displayRanking = "1.0 (1)";
    else if (ranking === "2") displayRanking = "2.0 (2)";
    else if (ranking === "3") displayRanking = "3.0 (3)";
    else if (ranking === "4") displayRanking = "4.0 (4)";
    else displayRanking = ranking;

    const fnegeLine = `FNEGE 2025: ${displayRanking}`;

    let archiveLocation = item.getField("archiveLocation") || "";

    // Remplace toute ligne FNEGE existante (2022, 2025 ou generique), sinon ajoute
    const fnegeRegex = /FNEGE(?:\s*2025|\s*2022)?\s*:\s?.*/;

    if (!fnegeRegex.test(archiveLocation)) {
      archiveLocation = (archiveLocation + "\n" + fnegeLine).trim();
    } else {
      archiveLocation = archiveLocation.replace(fnegeRegex, fnegeLine);
    }

    item.setField("archiveLocation", archiveLocation);
    await item.saveTx();

    console.log(`✔ ${journalRaw} -> ${fnegeLine}`);
    count++;
  }

  Zotero.alert(null, "FNEGE 2025", `✅ ${count} article(s) modifie(s) dans la selection`);
};

updateFNEGE2025InArchiveLocation();

</code></pre>
///// ARRETER DE COPIER AVANT CETTE LIGNE /////

4) cliquez sur Exécuter
5) rajouter la colonne Loc. in Archive

<img width="1850" height="207" alt="image" src="https://github.com/user-attachments/assets/c0602ddc-522c-434e-b02d-7a992f7fd5b4" />


Attention : il faut répéter cette action à chaque fois que vous voulez mettre à jour le classement FNEGE 2022 concernant vos articles. C'est manuel et non automatique.
