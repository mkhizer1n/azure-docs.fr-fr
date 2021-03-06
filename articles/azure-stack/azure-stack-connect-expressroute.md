---
title: Connexion d’Azure Stack à Azure à l’aide d’ExpressRoute
description: Découvrez comment connecter des réseaux virtuels dans Azure Stack à des réseaux virtuels dans Azure à l’aide d’ExpressRoute.
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/22/2018
ms.author: sethm
ms.reviewer: unknown
ms.lastreviewed: 10/22/2018
ms.openlocfilehash: b12e696260793a79bab6b384fc7be810eb2a803c
ms.sourcegitcommit: 898b2936e3d6d3a8366cfcccc0fccfdb0fc781b4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/30/2019
ms.locfileid: "55240826"
---
# <a name="connect-azure-stack-to-azure-using-azure-expressroute"></a>Connexion d’Azure Stack à Azure à l’aide d’Azure ExpressRoute

*S’applique à : systèmes intégrés Azure Stack et Kit de développement Azure Stack*

Cet article vous explique comment connecter un réseau virtuel Azure Stack à un réseau virtuel Azure à l’aide d’une connexion directe de [Microsoft Azure ExpressRoute](/azure/expressroute/).

Vous pouvez utiliser cet article sous forme de didacticiel et utiliser les exemples pour configurer le même environnement de test. Ou bien, vous pouvez utiliser l’article comme une procédure pas-à-pas qui vous guide à travers la configuration de votre propre environnement ExpressRoute.

## <a name="overview-assumptions-and-prerequisites"></a>Vue d’ensemble, hypothèses et conditions préalables

Azure ExpressRoute vous permet d’étendre vos réseaux locaux au cloud de Microsoft via une connexion privée assurée par un fournisseur de connectivité. ExpressRoute n’est pas une connexion VPN établie via le réseau Internet public.

Pour en savoir plus sur ExpressRoute, consultez la rubrique [Présentation d’ExpressRoute](../expressroute/expressroute-introduction.md).

### <a name="assumptions"></a>Hypothèses

Cet article suppose que vous avez :

* Une connaissance pratique de Azure.
* Une connaissance élémentaire de Azure Stack.
* Une connaissance élémentaire de la mise en réseau.

### <a name="prerequisites"></a>Prérequis

Pour connecter Azure Stack et Azure à l’aide d’ExpressRoute, vous devez tenir compte des exigences suivantes :

* Un [circuit ExpressRoute](../expressroute/expressroute-circuit-peerings.md) doit être configuré via un [fournisseur de connectivité](../expressroute/expressroute-locations.md).
* Vous devez disposer d’un abonnement Azure pour créer un circuit ExpressRoute et des réseaux virtuels dans Azure.
* Un routeur qui doit :
  * prendre en charge les connexions VPN de site à site entre son interface LAN et la passerelle multi-locataire d’Azure Stack.
  * être en mesure de créer plusieurs VRF (Virtual Routing and Forwarding) si plus d’un locataire est ajouté à votre déploiement Azure Stack.
* Un routeur qui possède :
  * Un port WAN connecté au circuit ExpressRoute.
  * Un port de réseau local connecté à la passerelle multi-locataire d’Azure Stack.

### <a name="expressroute-network-architecture"></a>Architecture réseau ExpressRoute

Le diagramme suivant montre les environnements Azure Stack et Azure une fois que vous avez configuré ExpressRoute à l’aide des exemples de cet article :

*Figure 1 : Réseau ExpressRoute*

![Réseau ExpressRoute](media/azure-stack-connect-expressroute/Conceptual.png)

Le schéma d’architecture suivant montre comment plusieurs locataires se connectent à Azure depuis l’infrastructure Azure Stack par le biais du routeur ExpressRoute au niveau de la périphérie Microsoft :

*Figure 2 : Connexions d’architecture mutualisées*

![Connexions d’architecture mutualisées avec ExpressRoute](media/azure-stack-connect-expressroute/Architecture.png)

L’exemple de cet article utilise la même architecture mutualisée illustrée dans la figure 2 pour connecter Azure Stack à Azure à l’aide de l’homologation privée ExpressRoute. La connexion s’effectue grâce à une connexion VPN de site à site établie entre la passerelle de réseau virtuel dans Azure Stack et un routeur ExpressRoute.

Les étapes de cet article vous expliquent comment créer une connexion de bout en bout entre deux réseaux virtuels, depuis deux locataires différents dans Azure Stack, et les réseaux virtuels correspondants dans Azure. Le paramétrage de deux locataires est facultatif, vous pouvez également utiliser ces étapes pour un seul locataire.

## <a name="configure-azure-stack"></a>Configurer Azure Stack

Pour configurer l’environnement Azure Stack pour le premier locataire, utilisez les étapes du diagramme suivant comme guide. Si vous configurez plusieurs locataires, répétez ces étapes :

>[!NOTE]
>Ces étapes expliquent comment créer des ressources à l’aide du portail Azure Stack, mais vous pouvez également utiliser PowerShell.

![Configuration réseau Azure Stack](media/azure-stack-connect-expressroute/image2.png)

### <a name="before-you-begin"></a>Avant de commencer

Avant de commencer à configurer Azure Stack, vous avez besoin :

* un déploiement de système intégré Azure Stack intégré ou un déploiement de Kit de développement Azure Stack (ASDK). Pour en savoir plus sur le déploiement du ASDK, consultez la rubrique relative au [démarrage rapide du déploiement du Kit de développement Azure Stack](azure-stack-deploy-overview.md).
* Une offre sur Azure Stack à laquelle votre utilisateur peut souscrire. Pour plus d’informations, consultez [Plans, offres et abonnements](azure-stack-plan-offer-quota-overview.md).

### <a name="create-network-resources-in-azure-stack"></a>Créer des ressources réseau dans Azure Stack

Suivez les procédures ci-dessous pour créer les ressources réseau nécessaires dans Azure Stack pour un locataire.

#### <a name="create-the-virtual-network-and-vm-subnet"></a>Créer le réseau virtuel et le sous-réseau de machine virtuelle

1. Connectez-vous au portail utilisateur avec un compte d’utilisateur (locataire).

2. Dans le portail, sélectionnez **+ Créer une ressource**.

3. Sous **Place de marché Azure**, sélectionnez **Mise en réseau**.

4. Sous **Sélection**, sélectionnez **Réseau virtuel**.

5. Sous **Créer un réseau virtuel**, entrez, au sein des champs appropriés, les valeurs indiquées dans le tableau suivant :

   |Champ  |Valeur  |
   |---------|---------|
   |NOM     |Tenant1VNet1         |
   |Espace d’adressage     |10.1.0.0/16|
   |Nom du sous-réseau     |Tenant1-Sub1|
   |Plage d’adresses de sous-réseau     |10.1.1.0/24|

6. Vous devriez voir l’abonnement que vous avez créé précédemment rempli dans le champ **Abonnement**. Pour les champs restants :

    * Sous **Groupe de ressources**, sélectionnez **Créer** pour créer un nouveau groupe de ressources ou, si vous en avez déjà un, sélectionnez **Utiliser l’existant**.
    * Vérifiez l’**emplacement** par défaut.
    * Cliquez sur **Créer**.
    * Cliquez sur **Épingler au tableau de bord** (facultatif).

#### <a name="create-the-gateway-subnet"></a>Créer le sous-réseau de passerelle

1. Sous **Réseau virtuel**, sélectionnez **Tenant1VNet1**.
1. Sous **PARAMÈTRES**, sélectionnez **Sous-réseaux**.
1. Sélectionnez **+ Sous-réseau de passerelle** pour ajouter un sous-réseau de passerelle au réseau virtuel.
1. Par défaut, le nom du sous-réseau est défini sur **GatewaySubnet**. Les sous-réseaux de passerelle sont des cas spéciaux et doivent utiliser ce nom pour fonctionner correctement.
1. Vérifiez que la **plage d’adresses** est **10.1.0.0/24**.
1. Cliquez sur **OK** pour créer le sous réseau de passerelle.

#### <a name="create-the-virtual-network-gateway"></a>Créer la passerelle de réseau virtuel

1. Dans le portail utilisateur Azure Stack, cliquez sur **+ Créer une ressource**.
1. Sous **Place de marché Azure**, sélectionnez **Mise en réseau**.
1. Sélectionnez **Passerelle de réseau virtuel** dans la liste des ressources réseau.
1. Dans le champ **Nom**, entrez **GW1**.
1. Sélectionnez **Réseau virtuel**.
1. Sélectionnez **Tenant1VNet1** dans la liste déroulante.
1. Sélectionnez **Adresse IP publique**, puis **Choisir une adresse IP publique**, cliquez ensuite sur **Créer**.
1. Dans le champ **Nom**, saisissez **GW1-PiP**, puis cliquez sur **OK**.
1. Par défaut, le **type de VPN** doit être défini sur **Basé sur un itinéraire**. Conservez ce paramètre.
1. Vérifiez que l’**abonnement** et l’**emplacement** sont corrects. Cliquez sur **Créer**.

#### <a name="create-the-local-network-gateway"></a>Créer la passerelle de réseau local

La ressource de passerelle de réseau local identifie la passerelle distante présente à l’autre extrémité de la connexion VPN. Pour cet exemple, l’extrémité distante de la connexion est la sous-interface LAN du routeur ExpressRoute. Pour le locataire 1, illustrée dans la figure 2, l’adresse distante est 10.60.3.255.

1. Connectez-vous au portail utilisateur Azure Stack avec votre compte d’utilisateur, puis sélectionnez **+ Créer une ressource**.
1. Sous **Place de marché Azure**, sélectionnez **Mise en réseau**.
1. Sélectionnez **Passerelle de réseau local** dans la liste des ressources.
1. Dans le champ **Nom**, saisissez **ER-Router-GW**.
1. Pour le champ **Adresse IP**, reportez-vous à la figure2. L’adresse IP de la sous-interface LAN du routeur ExpressRoute pour le locataire 1 est 10.60.3.255. Pour votre propre environnement, entrez l’adresse IP de l’interface correspondante de votre routeur.
1. Dans le champ **Espace d’adressage**, entrez l’espace d’adressage des réseaux virtuels auxquels se connecter dans Azure. Les sous-réseaux pour le locataire 1 dans la *Figure 2* sont les suivants :

   * 192.168.2.0/24 est le réseau virtuel hub dans Azure.
   * 10.100.0.0/16 est le réseau virtuel Spoke dans Azure.

   > [!IMPORTANT]
   > Cet exemple suppose que vous utilisiez des itinéraires statiques pour la connexion VPN de site à site entre la passerelle Azure Stack et le routeur ExpressRoute.

1. Vérifiez l’exactitude des valeurs des champs **Abonnement**, **Groupe de ressources** et **Emplacement**. Cliquez ensuite sur **Créer**.

#### <a name="create-the-connection"></a>Créer la connexion

1. Dans le portail utilisateur Azure Stack, sélectionnez **+ Créer une ressource**.
1. Sous **Place de marché Azure**, sélectionnez **Mise en réseau**.
1. Sélectionnez **Connexion** dans la liste des ressources.
1. Sous **De base**, choisissez **Site à site (IPSec)** comme **Type de connexion**.
1. Sélectionnez **Abonnement**, **Groupe de ressources** et **Emplacement**. Cliquez sur **OK**.
1. Sous **Paramètres**, sélectionnez **Passerelle de réseau virtuel**, puis **GW1**.
1. Sélectionnez **Passerelle de réseau local**, puis **ER Router GW**.
1. Dans le champ **Nom de la connexion**, entrez **ConnectToAzure**.
1. Dans le champ **Clé partagée (PSK)**, entrez **abc123**, puis sélectionnez **OK**.
1. Sous **Résumé**, sélectionnez **OK**.

#### <a name="get-the-virtual-network-gateway-public-ip-address"></a>Obtenir l’adresse IP publique de passerelle de réseau virtuel

Après avoir créé la passerelle de réseau virtuel, vous pouvez obtenir son adresse IP publique. Prenez note de cette adresse au cas où vous en avez besoin plus tard pour votre déploiement. Selon votre déploiement, cette adresse est utilisée comme **Adresse IP interne**.

1. Dans le portail utilisateur Azure Stack, sélectionnez **Toutes les ressources**.
1. Sous **toutes les ressources**, sélectionnez la passerelle de réseau virtuel, **GW1** dans l’exemple.
1. Sous **Passerelle de réseau virtuel**, sélectionnez **Vue d’ensemble** dans la liste des ressources. Vous pouvez également sélectionner **Propriétés**.
1. L’adresse IP que vous souhaitez noter est répertoriée sous **Adresse IP publique**. Pour l’exemple de configuration, cette adresse est 192.68.102.1.

#### <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Pour tester le trafic de données via la connexion VPN, vous avez besoin de machines virtuelles pour envoyer et recevoir des données dans le réseau virtuel d’Azure Stack. Créez une machine virtuelle et déployez-la vers le sous-réseau de machine virtuelle pour votre réseau virtuel.

1. Dans le portail utilisateur Azure Stack, sélectionnez **+ Créer une ressource**.
1. Sous **Place de marché Azure**, sélectionnez **Compute**.
1. Dans la liste des images de machine virtuelle, sélectionnez l’image **Windows Server 2016 Datacenter Evals**.

   >[!NOTE]
   >Si l’image utilisée pour cet article n’est pas disponible, demandez à votre opérateur Azure Stack de vous fournir une autre image Windows Server.

1. Dans le champ **Créer une machine virtuelle**, sélectionnez **Concepts de base**, puis saisissez **VM01** comme **Nom**.
1. Entrez un nom d’utilisateur et un mot de passe valides. Vous utiliserez ce compte pour vous connecter à la machine virtuelle une fois celle-ci créée.
1. Fournissez un **Abonnement**, un **Groupe de ressources** et un **Emplacement**. Sélectionnez **OK**.
1. Sous **Choisir une taille**, sélectionnez une taille de machine virtuelle pour cette instance, puis choisissez **Sélectionner**.
1. Sous **Paramètres**, vérifiez que :

   * Le réseau virtuel est **Tenant1VNet1**.
   * Le sous-réseau est défini sur **10.1.1.0/24**.

   Utilisez les paramètres par défaut et cliquez sur **OK**.

1. Sous **Résumé**, vérifiez la configuration de la machine virtuelle, puis cliquez sur **OK**.

Pour ajouter davantage de locataires, répétez les étapes suivies dans les sections suivantes :

* [Créer le réseau virtuel et le sous-réseau de machine virtuelle](#create-the-virtual-network-and-vm-subnet)
* [Créer le sous-réseau de passerelle](#create-the-gateway-subnet)
* [Créer la passerelle de réseau virtuel](#create-the-virtual-network-gateway)
* [Créer la passerelle de réseau local](#create-the-local-network-gateway)
* [Créer la connexion](#create-the-connection)
* [Créer une machine virtuelle](#create-a-virtual-machine)

Si vous prévoyez d’utiliser le locataire 2 en exemple, n’oubliez pas de modifier les adresses IP pour éviter les chevauchements.

### <a name="configure-the-nat-virtual-machine-for-gateway-traversal"></a>Configurer la machine virtuelle NAT pour la traversée de passerelle

> [!IMPORTANT]
> Cette section concerne uniquement les déploiements du Kit de développement Azure Stack (ASDK). La traduction d’adresses réseau (NAT) n’est pas nécessaire pour les déploiements à plusieurs nœuds.

Le Kit de développement Azure Stack est autonome et isolé du réseau sur lequel est déployé l’hôte physique. Le réseau VIP auquel les passerelles sont connectées n’est pas externe. Il est masqué derrière un routeur qui procède à la traduction d’adresses réseau (NAT).

Le routeur est une machine virtuelle Windows Server (AzS-BGPNAT01) qui exécute le rôle RRAS (Routing and Remote Access Services). Vous devez configurer la traduction d’adresses réseau sur la machine virtuelle AzS-BGPNAT01 pour permettre à la connexion VPN de site à site de se connecter aux deux extrémités.

#### <a name="configure-the-nat"></a>Configurer NAT

1. Connectez-vous à l’ordinateur hôte Azure Stack avec votre compte Administrateur.
1. Copiez et modifiez le script PowerShell suivant. Remplacez `"your administrator password"` par votre mot de passe administrateur, puis exécutez le script dans une fenêtre PowerShell ISE avec élévation de privilèges. Le script renvoie votre **adresse BGPNAT externe**.

   ```PowerShell
   cd \AzureStack-Tools-master\connect
   Import-Module .\AzureStack.Connect.psm1
   $Password = ConvertTo-SecureString "your administrator password" `
    -AsPlainText `
    -Force
   Get-AzureStackNatServerAddress `
    -HostComputer "azs-bgpnat01" `
    -Password $Password
   ```

1. Pour configurer le NAT, copiez et modifiez le script PowerShell suivant. Modifiez le script pour remplacer `'External BGPNAT address'` et `'Internal IP address'` par les valeurs d’exemple suivantes :

   * Pour *l’adresse BGPNAT externe*, utilisez 10.10.0.62
   * Pour *l’adresse IP interne*, utilisez 192.168.102.1

   Exécutez le script suivant depuis une session PowerShell ISE avec élévation de privilèges :

   ```PowerShell
   $ExtBgpNat = 'External BGPNAT address'
   $IntBgpNat = 'Internal IP address'

   # Designate the external NAT address for the ports that use the IKE authentication.
   Invoke-Command `
    -ComputerName azs-bgpnat01 `
     {Add-NetNatExternalAddress `
      -NatName BGPNAT `
      -IPAddress $Using:ExtBgpNat `
      -PortStart 499 `
      -PortEnd 501}
   Invoke-Command `
    -ComputerName azs-bgpnat01 `
     {Add-NetNatExternalAddress `
      -NatName BGPNAT `
      -IPAddress $Using:ExtBgpNat `
      -PortStart 4499 `
      -PortEnd 4501}
   # Create a static NAT mapping to map the external address to the Gateway public IP address to map the ISAKMP port 500 for PHASE 1 of the IPSEC tunnel.
   Invoke-Command `
    -ComputerName azs-bgpnat01 `
     {Add-NetNatStaticMapping `
      -NatName BGPNAT `
      -Protocol UDP `
      -ExternalIPAddress $Using:ExtBgpNat `
      -InternalIPAddress $Using:IntBgpNat `
      -ExternalPort 500 `
      -InternalPort 500}
   # Configure NAT traversal which uses port 4500 to  establish the complete IPSEC tunnel over NAT devices.
   Invoke-Command `
    -ComputerName azs-bgpnat01 `
     {Add-NetNatStaticMapping `
      -NatName BGPNAT `
      -Protocol UDP `
      -ExternalIPAddress $Using:ExtBgpNat `
      -InternalIPAddress $Using:IntBgpNat `
      -ExternalPort 4500 `
      -InternalPort 4500}

   ```

## <a name="configure-azure"></a>Configuration d’Azure

Après avoir terminé la configuration d’Azure Stack, vous pouvez déployer les ressources Azure. La figure suivante représente un exemple de réseau virtuel locataire dans Azure. Vous pouvez utiliser n’importe quel nom et schéma d’adressage pour désigner votre réseau virtuel dans Azure. Toutefois, la plage d’adresses des réseaux virtuels dans Azure et Azure Stack doit être unique et ne doit pas se chevaucher.

*Figure 3 : Réseaux virtuels Azure*

![Réseaux virtuels Azure](media/azure-stack-connect-expressroute/AzureArchitecture.png)

Les ressources que vous déployez dans Azure sont semblables aux ressources déployées dans Azure Stack. Vous allez déployer les composants suivants :

* Des réseaux virtuels et des sous-réseaux
* Un sous-réseau de passerelle
* Une passerelle de réseau virtuel
* Une connexion
* Un circuit ExpressRoute

L’infrastructure réseau Azure donnée en exemple est configurée de la façon suivante :

* Un modèle de réseau virtuel Hub (192.168.2.0/24) et Spoke (10.100.0.0./16) standard. Pour plus d’informations sur une topologie de réseau Hub et Spoke, consultez [Implémenter une topologie de réseau Hub et Spoke dans Azure](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke).
* Les charges de travail sont déployées dans le réseau virtuel Spoke et le circuit ExpressRoute est connecté au réseau virtuel Hub.
* Les deux réseaux virtuels sont connectés à l’aide de l’homologation de réseau virtuel.

### <a name="configure-the-azure-vnets"></a>Configurer les réseaux virtuels Azure

1. Connectez-vous au portail Azure avec vos informations d’identification Azure.
1. Créez le réseau virtuel Hub en utilisant la plage d’adresses 192.168.2.0/24.
1. Créez un sous-réseau à l’aide de la plage d’adresses 192.168.2.0/25 et ajoutez un sous-réseau de passerelle à l’aide de la plage d’adresses 192.168.2.128/27.
1. Créez le réseau virtuel Spoke et le sous-réseau en utilisant la plage d’adresses 10.100.0.0/16.

Pour en savoir plus sur la création de réseaux virtuels dans Azure, consultez la rubrique [Créer un réseau virtuel](../virtual-network/manage-virtual-network.md#create-a-virtual-network).

### <a name="configure-an-expressroute-circuit"></a>Configurer un circuit ExpressRoute

1. Passez en revue la configuration requise d’ExpressRoute dans [Configuration requise pour ExpressRoute et liste de contrôle](../expressroute/expressroute-prerequisites.md).

1. Suivez les étapes décrites dans la rubrique [Création et modification d’un circuit ExpressRoute](../expressroute/expressroute-howto-circuit-portal-resource-manager.md) pour créer un circuit ExpressRoute via votre abonnement Azure.

   >[!NOTE]
   >Donnez la clé de service pour votre circuit à votre service afin qu’ils puissent configurer votre circuit ExpressRoute à leur extrémité.

1. Suivez les étapes décrites dans la rubrique [Créer et modifier l’homologation pour un circuit ExpressRoute](../expressroute/expressroute-howto-routing-portal-resource-manager.md) pour configurer l’homologation privée sur le circuit ExpressRoute.

### <a name="create-the-virtual-network-gateway"></a>Créer la passerelle de réseau virtuel

Suivez les étapes décrites dans la rubrique [Configurer une passerelle de réseau virtuel pour ExpressRoute à l’aide de PowerShell](../expressroute/expressroute-howto-add-gateway-resource-manager.md) pour créer une passerelle de réseau virtuel pour ExpressRoute dans le réseau virtuel Hub.

### <a name="create-the-connection"></a>Créer la connexion

Pour connecter le circuit ExpressRoute au réseau virtuel Hub, suivez les étapes décrites dans [Connecter un réseau virtuel à un circuit ExpressRoute](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md).

### <a name="peer-the-vnets"></a>Homologuer les réseaux virtuels

Homologuez les réseaux virtuels Hub et Spoke en suivant les étapes décrites dans la rubrique [Créer une homologation de réseaux virtuels - Resource Manager - Même abonnement](../virtual-network/virtual-networks-create-vnetpeering-arm-portal.md). Lors de la configuration de VNet Peering, assurez-vous d’utiliser les options suivantes :

* De Hub à Spoke : **Autoriser le transit par passerelle**.
* De Spoke à Hub : **Utiliser la passerelle distante par défaut**.

### <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Déployez les machines virtuelles de votre charge de travail dans le réseau virtuel Spoke.

Répétez ces étapes pour connecter d’autres réseaux virtuels locataires dans Azure via leurs circuits ExpressRoute respectifs.

## <a name="configure-the-router"></a>Configuration du routeur

Vous pouvez utiliser le diagramme suivant de la configuration du routeur ExpressRoute comme guide pour configurer votre routeur ExpressRoute. Ce diagramme représente deux locataires (Tenant 1 et Tenant 2) avec leurs circuits ExpressRoute respectifs. Chaque locataire est connecté à son propre VRF (Virtual Routing and Forwarding) du côté LAN et WAN du routeur ExpressRoute. Cette configuration garantit l’isolation de bout en bout entre les deux clients. Notez les adresses IP utilisées dans les interfaces du routeur pendant que vous suivez l’exemple de configuration.

*Figure 4 : Configuration du routeur ExpressRoute*

![Configuration du routeur ExpressRoute](media/azure-stack-connect-expressroute/EndToEnd.png)

Vous pouvez utiliser n’importe quel routeur qui prend en charge le VPN IKEv2 et BGP pour mettre fin à la connexion VPN de site à site établie depuis Azure Stack. Le même routeur est utilisé pour se connecter à Azure via un circuit ExpressRoute.

L’exemple de configuration du routeur des services d’agrégation Cisco ASR 1000 ci-après prend en charge l’infrastructure réseau représentée dans le diagramme *Configuration du routeur ExpressRoute*.

```
ip vrf Tenant 1
 description Routing Domain for PRIVATE peering to Azure for Tenant 1
 rd 1:1
!
ip vrf Tenant 2
 description Routing Domain for PRIVATE peering to Azure for Tenant 2
 rd 1:5
!
crypto ikev2 proposal V2-PROPOSAL2
description IKEv2 proposal for Tenant 1
encryption aes-cbc-256
 integrity sha256
 group 2
crypto ikev2 proposal V4-PROPOSAL2
description IKEv2 proposal for Tenant 2
encryption aes-cbc-256
 integrity sha256
 group 2
!
crypto ikev2 policy V2-POLICY2
description IKEv2 Policy for Tenant 1
match fvrf Tenant 1
 match address local 10.60.3.255
 proposal V2-PROPOSAL2
description IKEv2 Policy for Tenant 2
crypto ikev2 policy V4-POLICY2
 match fvrf Tenant 2
 match address local 10.60.3.251
 proposal V4-PROPOSAL2
!
crypto ikev2 profile V2-PROFILE
description IKEv2 profile for Tenant 1
match fvrf Tenant 1
 match address local 10.60.3.255
 match identity remote any
 authentication remote pre-share key abc123
 authentication local pre-share key abc123
 ivrf Tenant 1
!
crypto ikev2 profile V4-PROFILE
description IKEv2 profile for Tenant 2
 match fvrf Tenant 2
 match address local 10.60.3.251
 match identity remote any
 authentication remote pre-share key abc123
 authentication local pre-share key abc123
 ivrf Tenant 2
!
crypto ipsec transform-set V2-TRANSFORM2 esp-gcm 256
 mode tunnel
crypto ipsec transform-set V4-TRANSFORM2 esp-gcm 256
 mode tunnel
!
crypto ipsec profile V2-PROFILE
 set transform-set V2-TRANSFORM2
 set ikev2-profile V2-PROFILE
!
crypto ipsec profile V4-PROFILE
 set transform-set V4-TRANSFORM2
 set ikev2-profile V4-PROFILE
!
interface Tunnel10
description S2S VPN Tunnel for Tenant 1
 ip vrf forwarding Tenant 1
 ip address 11.0.0.2 255.255.255.252
 ip tcp adjust-mss 1350
 tunnel source TenGigabitEthernet0/1/0.211
 tunnel mode ipsec ipv4
 tunnel destination 10.10.0.62
 tunnel vrf Tenant 1
 tunnel protection ipsec profile V2-PROFILE
!
interface Tunnel20
description S2S VPN Tunnel for Tenant 2
 ip vrf forwarding Tenant 2
 ip address 11.0.0.2 255.255.255.252
 ip tcp adjust-mss 1350
 tunnel source TenGigabitEthernet0/1/0.213
 tunnel mode ipsec ipv4
 tunnel destination 10.10.0.62
 tunnel vrf VNET3
 tunnel protection ipsec profile V4-PROFILE
!
interface GigabitEthernet0/0/1
 description PRIMARY ExpressRoute Link to AZURE over Equinix
 no ip address
 negotiation auto
!
interface GigabitEthernet0/0/1.100
description Primary WAN interface of Tenant 1
 description PRIMARY ER link supporting Tenant 1 to Azure
 encapsulation dot1Q 101
 ip vrf forwarding Tenant 1
 ip address 192.168.1.1 255.255.255.252
!
interface GigabitEthernet0/0/1.102
description Primary WAN interface of Tenant 2
 description PRIMARY ER link supporting Tenant 2 to Azure
 encapsulation dot1Q 102
 ip vrf forwarding Tenant 2
 ip address 192.168.1.17 255.255.255.252
!
interface GigabitEthernet0/0/2
 description BACKUP ExpressRoute Link to AZURE over Equinix
 no ip address
 negotiation auto
!
interface GigabitEthernet0/0/2.100
description Secondary WAN interface of Tenant 1
 description BACKUP ER link supporting Tenant 1 to Azure
 encapsulation dot1Q 101
 ip vrf forwarding Tenant 1
 ip address 192.168.1.5 255.255.255.252
!
interface GigabitEthernet0/0/2.102
description Secondary WAN interface of Tenant 2
description BACKUP ER link supporting Tenant 2 to Azure
 encapsulation dot1Q 102
 ip vrf forwarding Tenant 2
 ip address 192.168.1.21 255.255.255.252
!
interface TenGigabitEthernet0/1/0
 description Downlink to ---Port 1/47
 no ip address
!
interface TenGigabitEthernet0/1/0.211
 description LAN interface of Tenant 1
description Downlink to --- Port 1/47.211
 encapsulation dot1Q 211
 ip vrf forwarding Tenant 1
 ip address 10.60.3.255 255.255.255.254
!
interface TenGigabitEthernet0/1/0.213
description LAN interface of Tenant 2
 description Downlink to --- Port 1/47.213
 encapsulation dot1Q 213
 ip vrf forwarding Tenant 2
 ip address 10.60.3.251 255.255.255.254
!
router bgp 65530
 bgp router-id <removed>
 bgp log-neighbor-changes
 description BGP neighbor config and route advertisement for Tenant 1 VRF
 address-family ipv4 vrf Tenant 1
  network 10.1.0.0 mask 255.255.0.0
  network 10.60.3.254 mask 255.255.255.254
  network 192.168.1.0 mask 255.255.255.252
  network 192.168.1.4 mask 255.255.255.252
  neighbor 10.10.0.62 remote-as 65100
  neighbor 10.10.0.62 description VPN-BGP-PEER-for-Tenant 1
  neighbor 10.10.0.62 ebgp-multihop 5
  neighbor 10.10.0.62 activate
  neighbor 10.60.3.254 remote-as 4232570301
  neighbor 10.60.3.254 description LAN peer for CPEC:INET:2112 VRF
  neighbor 10.60.3.254 activate
  neighbor 10.60.3.254 route-map BLOCK-ALL out
  neighbor 192.168.1.2 remote-as 12076
  neighbor 192.168.1.2 description PRIMARY ER peer for Tenant 1 to Azure
  neighbor 192.168.1.2 ebgp-multihop 5
  neighbor 192.168.1.2 activate
  neighbor 192.168.1.2 soft-reconfiguration inbound
  neighbor 192.168.1.2 route-map Tenant 1-ONLY out
  neighbor 192.168.1.6 remote-as 12076
  neighbor 192.168.1.6 description BACKUP ER peer for Tenant 1 to Azure
  neighbor 192.168.1.6 ebgp-multihop 5
  neighbor 192.168.1.6 activate
  neighbor 192.168.1.6 soft-reconfiguration inbound
  neighbor 192.168.1.6 route-map Tenant 1-ONLY out
  maximum-paths 8
 exit-address-family
 !
description BGP neighbor config and route advertisement for Tenant 2 VRF
address-family ipv4 vrf Tenant 2
  network 10.1.0.0 mask 255.255.0.0
  network 10.60.3.250 mask 255.255.255.254
  network 192.168.1.16 mask 255.255.255.252
  network 192.168.1.20 mask 255.255.255.252
  neighbor 10.10.0.62 remote-as 65300
  neighbor 10.10.0.62 description VPN-BGP-PEER-for-Tenant 2
  neighbor 10.10.0.62 ebgp-multihop 5
  neighbor 10.10.0.62 activate
  neighbor 10.60.3.250 remote-as 4232570301
  neighbor 10.60.3.250 description LAN peer for CPEC:INET:2112 VRF
  neighbor 10.60.3.250 activate
  neighbor 10.60.3.250 route-map BLOCK-ALL out
  neighbor 192.168.1.18 remote-as 12076
  neighbor 192.168.1.18 description PRIMARY ER peer for Tenant 2 to Azure
  neighbor 192.168.1.18 ebgp-multihop 5
  neighbor 192.168.1.18 activate
  neighbor 192.168.1.18 soft-reconfiguration inbound
  neighbor 192.168.1.18 route-map VNET-ONLY out
  neighbor 192.168.1.22 remote-as 12076
  neighbor 192.168.1.22 description BACKUP ER peer for Tenant 2 to Azure
  neighbor 192.168.1.22 ebgp-multihop 5
  neighbor 192.168.1.22 activate
  neighbor 192.168.1.22 soft-reconfiguration inbound
  neighbor 192.168.1.22 route-map VNET-ONLY out
  maximum-paths 8
 exit-address-family
!
ip forward-protocol nd
!
ip as-path access-list 1 permit ^$
ip route vrf Tenant 1 10.1.0.0 255.255.0.0 Tunnel10
ip route vrf Tenant 2 10.1.0.0 255.255.0.0 Tunnel20
!
ip prefix-list BLOCK-ALL seq 5 deny 0.0.0.0/0 le 32
!
route-map BLOCK-ALL permit 10
 match ip address prefix-list BLOCK-ALL
!
route-map VNET-ONLY permit 10
 match as-path 1
!
```

## <a name="test-the-connection"></a>Tester la connexion

Lorsque la connexion de site à site et le circuit ExpressRoute sont établis, testez votre connexion.

Effectuez les tests ping suivants :

* Inscrivez-vous à l’une des machines virtuelles dans votre réseau virtuel Azure et effectuez un test ping sur la machine virtuelle créée dans Azure Stack.
* Inscrivez-vous à l’une des machines virtuelles que vous avez créées dans Azure Stack et effectuez un test ping sur la machine virtuelle créée dans le réseau virtuel Azure.

>[!NOTE]
>Pour vérifier que vous envoyez bien le trafic via les connexions site à site et ExpressRoute, vous devez effectuer un test ping aux deux extrémités avec l’adresse IP dédiée de la machine virtuelle, et non avec son adresse IP virtuelle.

### <a name="allow-icmp-in-through-the-firewall"></a>Autoriser la transmission du protocole ICMP via le pare-feu

Par défaut, Windows Server 2016 n’autorise pas la transmission des paquets ICMP entrants via le pare-feu. Pour chaque machine virtuelle que vous utilisez pour des tests ping, vous devez autoriser les paquets ICMP entrants. Pour créer une règle de pare-feu pour ICMP, exécutez l’applet de commande suivante dans une fenêtre PowerShell avec élévation de privilèges :

```PowerShell
# Create ICMP firewall rule.
New-NetFirewallRule `
  –DisplayName “Allow ICMPv4-In” `
  –Protocol ICMPv4
```

### <a name="ping-the-azure-stack-virtual-machine"></a>Effectuer un test ping sur la machine virtuelle Azure Stack

1. Connectez-vous au portail utilisateur Azure Stack avec un compte locataire.

1. Trouvez la machine virtuelle créée et sélectionnez-la.

1. Sélectionnez **Connecter**.

1. À partir d’une invite de commandes Windows ou de PowerShell avec élévation de privilèges, entrez **ipconfig/all**. Notez l’adresse IPv4 retournée en sortie.

1. Effectuez un test ping de cette adresse IPv4 à partir de la machine virtuelle dans le réseau virtuel Azure.

   Dans l’environnement représenté en exemple, l’adresse IPv4 provient du sous-réseau 10.1.1.x/24. Dans votre environnement, l’adresse peut être différente. Toutefois, elle doit se trouver dans le sous-réseau que vous avez créé pour le sous-réseau du réseau virtuel locataire.

### <a name="view-data-transfer-statistics"></a>Afficher les statistiques de transfert de données

Si vous souhaitez connaître le volume de trafic qui transite via votre connexion, accédez au portail utilisateur Azure Stack. C’est également un bon moyen de savoir si les données de votre test ping sont passées au travers des connexions VPN et ExpressRoute :

1. Connectez-vous au portail utilisateur Azure Stack avec votre compte locataire et sélectionnez **Toutes les ressources**.
1. Accédez au groupe de ressources pour lequel votre passerelle VPN a été créée et sélectionnez le type d’objet **Connexions**.
1. Sélectionnez la connexion **ConnectToAzure** dans la liste.
1. Dans **Connexions**>**Vue d’ensemble**, vous pouvez visualiser les statistiques de **données entrantes** et de **données sortantes**. Vous devriez voir des valeurs non nulles.

   ![Données entrantes et Données sortantes](media/azure-stack-connect-expressroute/DataInDataOut.png)

## <a name="next-steps"></a>Étapes suivantes

[Déployer des applications sur Azure et Azure Stack](azure-stack-solution-pipeline.md)
