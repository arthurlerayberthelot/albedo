# TODO

CONTINUOUS IMPROVEMENTS :

* Refactoring du code de base pour résilience et optimisation des performances
* Refactoring du code pour simplicité, lisibilité et maintenabilité
* Revue de sécurité

NEEDED :

* SYNC fix : weird behaviour/latency when syncing (pushing) after note creation : it pulls and erase new notes instead of pushing new notes on gist. Maybe get rid of autopush (not really needed, added complexity)
* New folder feature : "/folder" from the commander create a folder "folder". "/" is both the new command symbol AND "root" folder (where everything is). "/" is displayed by default (top but where?). When switching to "/folder" (typing "/folder" in the commander), view change to all children of "/folder". When creating a block in "/folder", it becomes child of parent folder. It is not a new layer to "?".

TO BE CONSIDERED :

* Mobile integration : how ? PWA (with free github page) ?

---

IDENTIFIED ISSUES (from code review) :

