Erkläre Pull Request #$ARGUMENTS so einfach wie möglich.

Die PR-Nummer wird über $ARGUMENTS übergeben.

Wenn $ARGUMENTS leer ist, frage nach der PR-Nummer.

Nutze das GitHub CLI, um den PR zu lesen:

gh pr view $ARGUMENTS --json number,title,author,body,url,state,baseRefName,headRefName,files,commits,comments,reviews
gh pr diff $ARGUMENTS

Wenn `gh` nicht funktioniert, erkläre kurz das Problem und frage nach PR-Link oder Diff.

Antworte in dieser Struktur:

1. Kurz gesagt:
Was macht dieser PR in 1-2 einfachen Sätzen?

2. Wichtigste Änderungen:
Liste die wichtigsten Änderungen in einfachen Worten auf.

3. Warum ist das nützlich?
Erkläre den Zweck des PR.

4. Worauf sollte man achten?
Nenne mögliche Risiken, offene Fragen oder Dinge zum Testen.

Regeln:
- Schreibe einfach und verständlich.
- Halte dich kurz.
- Vermeide unnötige Fachbegriffe.
- Wenn du Fachbegriffe nutzt, erkläre sie kurz.
- Nutze nur Informationen aus `gh pr view` und `gh pr diff`.
- Erfinde keine Details, die nicht im PR stehen.

