# Chapitre 5 -- Limites et perimetre de ce parcours

Ce parcours couvre la presentation generale du depot et sa relation de
dependance forte avec les patrons `op-service` du monorepo Optimism,
l orchestration du cycle de vie dans le paquet `service/` (demarrage
ordonne, arret tolerant aux erreurs partielles), les trois contrats
d extension (`Config`, `Driver`, `Metricer`) que tout service concret doit
satisfaire, et l implementation de reference `example/` qui illustre
chacun de ces contrats avec un driver de sondage L1 et une API RPC
minimale.

Sont volontairement laisses hors champ : le detail interne des paquets
`op-service` importes (`cliapp`, `httputil`, `oppprof`, `oprpc`, `txmgr`,
etc.), qui appartiennent au monorepo Optimism et non a ce depot ; le
contenu de `example/config/config.go`, `example/flags/flags.go` et
`example/metrics/metrics.go` au-dela de leur role mentionne au chapitre 4
(definition des flags CLI concrets et des metriques specifiques a
l exemple, qui suivent des patrons standard d op-service non repetes ici) ;
et le `Makefile` qui pilote la compilation et l injection des variables de
version.

L objectif reste le meme que pour les parcours precedents de cette
bibliotheque : comprendre precisement comment ce squelette factorise le
cycle de vie et l observabilite communs a un service backend Base, sans
pretendre couvrir l integralite des bibliotheques `op-service` sur
lesquelles il s appuie.
