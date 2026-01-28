1) Selectionner votre bibliothèque puis sélectionner tous les articles de la bibliothèque (ctrl + a) dont vous souhaitez parser et supprimer la plausible présence d'affiliation université dans les métadonnées auteur du fichier parent
2) Cliquer sur outil --> développeur --> Run JavaScript
![image](https://github.com/user-attachments/assets/b42d3919-ae3a-4b99-bdf7-3b529fbabeb2)

3) copier tout le code suivant en dessous de code : 

///// COMMENCER A COPIER APRES CETTE LIGNE /////

<pre><code>

// Zotero 8 - suppression DEFINITIVE des auteurs contenant "university"
// Detection ultra-robuste (inspecte tout le contenu du creator)

(async () => {
  const KEYWORD = "university";
  const zp = Zotero.getActiveZoteroPane();

  if (!zp) {
    Zotero.alert(null, "Erreur", "ZoteroPane introuvable.");
    return;
  }

  let items = zp.getSelectedItems();
  if (!items || !items.length) {
    Zotero.alert(null, "Selection requise", "Selectionne au moins un item parent.");
    return;
  }

  // Parents uniquement
  items = items.filter(i => i && i.isRegularItem?.() && !i.parentItemID);

  let itemsModified = 0;
  let creatorsRemoved = 0;

  for (const item of items) {
    const creators = item.getCreators();
    if (!creators?.length) continue;

    const kept = [];
    let removedHere = 0;

    for (const c of creators) {
      // 🔥 Detection SUR TOUTES les proprietes du creator
      const rawText = JSON.stringify(c).toLowerCase();

      if (rawText.includes(KEYWORD)) {
        removedHere++; // suppression COMPLETE du champ Auteur
      } else {
        kept.push(c);
      }
    }

    if (removedHere > 0) {
      item.setCreators(kept);
      await item.saveTx();
      itemsModified++;
      creatorsRemoved += removedHere;
    }
  }

  Zotero.alert(
    null,
    "Nettoyage termine",
    `Items modifies : ${itemsModified}
Champs Auteur supprimes : ${creatorsRemoved}
Mot-cle detecte : "${KEYWORD}" (insensible a la casse)`
  );
})();

</code></pre>
///// ARRETER DE COPIER AVANT CETTE LIGNE /////

4) cliquez sur Exécuter

