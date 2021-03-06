# Vue d’ensemble
## [À propos de la mise en réseau Azure](networking-overview.md)
## Architecture
### [Centres de données virtuels](/azure/architecture/vdc/networking-virtual-datacenter)
### [Routage asymétrique avec plusieurs chemins d’accès réseau](../expressroute/expressroute-asymmetric-routing.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Conceptions de réseau sécurisé](../best-practices-network-security.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Topologie hub-and-spoke](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)
### [Meilleures pratiques en matière de sécurité réseau](../security/azure-security-network-security-best-practices.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Appliances virtuelles réseau hautement disponible](https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/nva-ha )
### [Combiner les méthodes d’équilibrage de charge](../traffic-manager/traffic-manager-load-balancing-azure.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Récupération d’urgence à l’aide d’Azure DNS et Traffic Manager](disaster-recovery-dns-traffic-manager.md)
## Planifier et concevoir
### [Réseaux virtuels](../virtual-network/virtual-network-vnet-plan-design-arm.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Connectivité intersite - VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Options de connectivité intersite - connexion privée dédiée](../expressroute/expressroute-workflows.md?toc=%2fazure%2fnetworking%2ftoc.json)
### Interopérabilité de connectivité du serveur principal
#### [Préface et initialisation (tearDown) de test](connectivty-interoperability-preface.md?toc=%2fazure%2fnetworking%2ftoc.json)
#### [Configuration de l’initialisation (tearDown) de test](connectivty-interoperability-configuration.md?toc=%2fazure%2fnetworking%2ftoc.json)
#### [Analyse du plan de contrôle](connectivty-interoperability-control-plane.md?toc=%2fazure%2fnetworking%2ftoc.json)
#### [Analyse du plan de données](connectivty-interoperability-data-plane.md?toc=%2fazure%2fnetworking%2ftoc.json)

##  Concepts
### [Réseaux virtuels](../virtual-network/virtual-networks-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Équilibrage de charge réseau](../load-balancer/load-balancer-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Équilibrage de charge application](../application-gateway/overview.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [DNS](../dns/dns-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Distribution du trafic DNS](../traffic-manager/traffic-manager-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Se connecter en local - VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Se connecter en local - dédié](../expressroute/expressroute-introduction.md?toc=%2fazure%2fnetworking%2ftoc.json)


# Prise en main
## [Créer votre premier réseau virtuel](../virtual-network/quick-create-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)

# Procédures
## Connectivité Internet
### [Équilibrage de charge réseau public serveurs](../load-balancer/load-balancer-get-started-internet-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Serveurs publics application charge solde](../application-gateway/quick-create-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Protéger les applications web](../application-gateway/application-gateway-web-application-firewall-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Répartir le trafic entre les emplacements](../traffic-manager/traffic-manager-configure-geographic-routing-method.md?toc=%2fazure%2fnetworking%2ftoc.json)
## Connectivité interne
### [Équilibrage de charge réseau privés serveurs](../load-balancer/load-balancer-get-started-ilb-arm-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Serveurs d’application charge solde privés](../application-gateway/application-gateway-ilb-arm.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Connecter des réseaux virtuels (même emplacement)](../virtual-network/virtual-networks-create-vnetpeering-arm-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Connecter des réseaux virtuels (emplacements)](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
## Connectivité intersite
### [Créer une connexion VPN S2S (IPsec/IKE)](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Créer une connexion VPN P2S (SSTP avec certificats)](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Créer une connexion privée dédiée (ExpressRoute)](../expressroute/expressroute-howto-circuit-portal-resource-manager.md?toc=%2fazure%2fnetworking%2ftoc.json)

## gestion
### [Vue d’ensemble de la surveillance réseau](network-monitoring-overview.md)
### [Vérifier l’utilisation des ressources par rapport aux limites de Azure](check-usage-against-limits.md)
### [Topologie du réseau](../network-watcher/network-watcher-topology-powershell.md?toc=%2fazure%2fnetworking%2ftoc.json)

## Exemples de scripts
### [Interface de ligne de commande Azure](cli-samples.md)
### [Azure PowerShell](powershell-samples.md)

## Tutoriels
### [Équilibrer la charge des machines virtuelles](../virtual-machines/linux/tutorial-load-balancer.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [Connecter un ordinateur à un réseau virtuel](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)

# Informations de référence
## [Interface de ligne de commande Azure](https://docs.microsoft.com/cli/azure/network)
## [Azure PowerShell](https://docs.microsoft.com/powershell/module/azurerm.network/?view=azurermps-3.8.0)
## [.Net](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.network?view=azuremgmtnetwork-9.1.0-preview)
## [Node.JS](https://azure.microsoft.com/develop/nodejs/#azure-sdk)
## [REST](https://msdn.microsoft.com/library/mt163658.aspx)

# Ressources
## [Créer des modèles](/azure/azure-resource-manager/resource-group-authoring-templates?toc=%2fazure%2fnetworking%2ftoc.json)
## [Feuille de route Azure](https://azure.microsoft.com/roadmap/?category=networking)
## [Modèles fournis par la communauté](https://azure.microsoft.com/resources/templates/)
## [Blog sur la mise en réseau](https://azure.microsoft.com/blog/topics/networking)
## [Tarification](https://azure.microsoft.com/pricing)
## [Calculatrice de prix](https://azure.microsoft.com/pricing/calculator/)
## [Disponibilité régionale](https://azure.microsoft.com/regions/services/)
## [Stack Overflow](http://stackoverflow.com/questions/tagged/azure-virtual-network)
## [Vidéos](https://azure.microsoft.com/resources/videos/index/?services=virtual-network)

