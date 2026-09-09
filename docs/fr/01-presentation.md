# Chapitre 1 -- Presentation de framework

Ce depot est volontairement minuscule (trois commits au total au moment de
la lecture) : c est un squelette Go generique pour construire des services
backend chez Base, explicitement decrit dans le README comme s appuyant
fortement sur les patrons de conception du monorepo `optimism`
(`github.com/ethereum-optimism/optimism/op-service`). Plutot que
d imposer un cadre applicatif complet, ce depot factorise l infrastructure
commune a tout service backend d une equipe habituee a l ecosysteme OP
Stack : cycle de vie (demarrage/arret gracieux), configuration en ligne de
commande, metriques Prometheus, profilage pprof, et serveur JSON-RPC.

Le code source se divise en exactement deux paquets Go. `service/` (le
"framework" proprement dit) definit quatre interfaces generiques --
`Service`, `Config`, `Driver`, `Metricer` -- sans aucune logique metier :
c est le contrat que tout service concret doit satisfaire pour beneficier
de l orchestration commune. `example/` est une implementation complete et
fonctionnelle de ce contrat : un service qui interroge periodiquement la
tete de chaine L1 et expose un point d entree JSON-RPC pour soumettre des
transactions -- suffisamment simple pour servir de modele copiable a un
nouveau service, mais suffisamment complet pour illustrer chaque piece du
contrat `service/`.

Cette repartition en deux paquets (framework generique + exemple concret
non couple) est elle-meme la principale valeur pedagogique du depot : elle
montre a un nouvel ingenieur Base exactement quelles pieces il doit
implementer (`Config`, `Driver`, eventuellement une API RPC et un
`Metricer`) pour brancher un service maison sur l infrastructure commune
d observabilite et de cycle de vie.
