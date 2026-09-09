# Chapitre 2 -- Service : orchestration et cycle de vie

`service.Service` est l interface centrale du paquet : elle embarque
`cliapp.Lifecycle` (le contrat standard `optimism/op-service` pour un
composant demarrable/arretable en ligne de commande), ajoute `Kill()`
(arret force, non gracieux), et quatre methodes de construction fluides --
`WithDriver`, `WithMetrics`, `WithRPC`, `WithRPCs` -- qui retournent
`Service` a chaque appel, permettant un style de configuration chaine
(`service.NewService(...).WithDriver(d).WithMetrics(m).WithRPC(api)`), tel
qu illustre dans `example/cmd/main.go` (chapitre 4).

L implementation concrete `*service` compose un `driver` (optionnel), un
`metrics` (optionnel), une liste de `rpc.API` (optionnelle), et trois
sous-services internes geres automatiquement : un service de profilage
pprof (`oppprof.Service`), un serveur HTTP de metriques
(`httputil.HTTPServer`), et un serveur JSON-RPC (`oprpc.Server`). La
methode `Start` orchestre leur demarrage dans un ordre precis et
deliberement choisi : d abord les metriques (si configurees), puis le
profilage pprof (toujours), puis le serveur RPC (seulement si des `rpc.API`
ont ete enregistrees), et enfin le `driver` -- de sorte que l
observabilite (metriques, profilage) soit deja active avant que la logique
metier ne commence a s executer, et que les compteurs `RecordInfo` /
`RecordUp` ne soient publies qu une fois tout le reste demarre avec
succes.

`Stop` suit une logique symetrique mais tolerante aux erreurs partielles :
elle tente d arreter le `driver`, le serveur RPC, le service pprof, puis
le serveur de metriques, en accumulant toute erreur rencontree via
`errors.Join` plutot que de s arreter au premier echec -- garantissant que
chaque sous-composant a au moins une chance d etre arrete proprement meme
si un autre a echoue. Un drapeau atomique (`atomic.Bool`) empeche un
double arret et retourne `ErrAlreadyStopped` si `Stop` est appelee apres
un arret deja reussi.
