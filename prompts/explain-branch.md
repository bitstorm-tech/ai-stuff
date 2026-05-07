Erkläre alle Änderungen im aktuell ausgecheckten Branch so einfach wie möglich.

$ARGUMENTS ist optional und kann den Basis-Branch enthalten, zum Beispiel:

/branch-erklaeren main
/branch-erklaeren develop

Wenn $ARGUMENTS leer ist, versuche den Basis-Branch automatisch zu erkennen:
- zuerst den Default-Branch von origin
- sonst origin/main
- sonst origin/master

Nutze Git, um die Änderungen zu lesen:

git branch --show-current
git status --short --branch
git symbolic-ref refs/remotes/origin/HEAD --short
git log --oneline <BASIS_BRANCH>..HEAD
git diff --stat <BASIS_BRANCH>...HEAD
git diff <BASIS_BRANCH>...HEAD

Wenn der Basis-Branch nicht eindeutig ist, frage nach dem Basis-Branch.

Antworte in dieser Struktur:

1. Kurz gesagt:
Was ändert dieser Branch in 1-2 einfachen Sätzen?

2. Wichtigste Änderungen:
Liste die wichtigsten Änderungen in einfachen Worten auf.

3. Warum ist das nützlich?
Erkläre den Zweck der Änderungen.

4. Worauf sollte man achten?
Nenne mögliche Risiken, offene Fragen oder Dinge zum Testen.

Regeln:
- Schreibe einfach und verständlich.
- Halte dich kurz.
- Vermeide unnötige Fachbegriffe.
- Wenn du Fachbegriffe nutzt, erkläre sie kurz.
- Nutze nur Informationen aus `git log`, `git status` und `git diff`.
- Erfinde keine Details, die nicht aus den Änderungen hervorgehen.

