---
title: 'Azure バックエンド接続機能の相互運用性: テスト セットアップ | Microsoft Docs'
description: この記事では、Azure における ExpressRoute、サイト間 VPN、仮想ネットワーク ピアリングの相互運用性を分析する際に使用できるテスト セットアップについて説明します。
documentationcenter: na
services: networking
author: rambk
manager: tracsman
ms.service: expressroute,vpn-gateway,virtual-network
ms.topic: article
ms.workload: infrastructure-services
ms.date: 10/18/2018
ms.author: rambala
ms.openlocfilehash: bded5dbf0084d230997be178c1f9a7b8a184ac07
ms.sourcegitcommit: b62f138cc477d2bd7e658488aff8e9a5dd24d577
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/13/2018
ms.locfileid: "51613231"
---
# <a name="interoperability-in-azure-back-end-connectivity-features-test-setup"></a>Azure バックエンド接続機能の相互運用性: テスト セットアップ

この記事では、Azure のネットワーク サービスがコントロール プレーン レベルとデータ プレーン レベルでどのように相互運用されるかを分析するために使用できる、テスト セットアップについて説明します。 Azure のネットワーク コンポーネントについて簡単に見てみましょう。

-   **Azure ExpressRoute**: Azure ExpressRoute でプライベート ピアリングを使用すると、ご利用の Azure 仮想ネットワーク デプロイにオンプレミス ネットワーク内のプライベート IP 空間を直接接続することができます。 ExpressRoute は、より帯域幅の広いプライベート接続を実現するのに役立ちます。 ExpressRoute 接続は、多くの ExpressRoute エコ パートナーから SLA 付きで提供されています。 ExpressRoute とその構成方法について詳しくは、「[ExpressRoute の概要][ExpressRoute]」をご覧ください。
-   **サイト間 VPN**: Azure VPN Gateway をサイト間 VPN として使用すると、オンプレミス ネットワークをインターネットまたは ExpressRoute 経由で Azure に安全に接続することができます。 サイト間 VPN を Azure に接続するように構成する方法については、[VPN Gateway の構成][VPN]に関する記事をご覧ください。
-   **VNet ピアリング**: 仮想ネットワーク (VNet) ピアリングを使用すると、Azure Virtual Network 内の VNet 間の接続を確立できます。 VNet ピアリングについて詳しくは、[VNet ピアリングに関するチュートリアル][VNet]をご覧ください。

## <a name="test-setup"></a>テストの設定

次の図は、テスト セットアップを示したものです。

[![1]][1]

テスト セットアップの中心となるのは、Azure Region 1 のハブ VNet です。 ハブ VNet は、次の方法で各種ネットワークに接続されています。

-   ハブ VNet は、VNet ピアリングを使用してスポーク VNet に接続されます。 スポーク VNet は、ハブ VNet 内の両方のゲートウェイにリモート アクセスできます。
-   ハブ VNet は、サイト間 VPN を使用してブランチ VNet に接続されます。 この接続では eBGP を使用してルートを交換します。
-   ハブ VNet は、ExpressRoute プライベート ピアリングをプライマリ パスに使用して、オンプレミスの Location 1 ネットワークに接続されます。 バックアップ パスにはサイト間 VPN 接続が使用されます。 以降、この記事では、この ExpressRoute 回線を ExpressRoute 1 と呼びます。 ExpressRoute 回線には、冗長接続による高可用性が既定で確保されています。 ExpressRoute 1 では、セカンダリの Microsoft Enterprise Edge Router (MSEE) に接続するセカンダリ カスタマー エッジ (CE) ルーターのサブインターフェイスが無効になります。 上の図では、二重線の矢印にある赤い線が、無効になった CE ルーターのサブインターフェイスを表しています。
-   ハブ VNet は、別の ExpressRoute プライベート ピアリングを使用して、オンプレミスの Location 2 ネットワークに接続されます。 以降、この記事では、この 2 つ目の ExpressRoute 回線を ExpressRoute 2 と呼びます。
-   また、ハブ VNet とオンプレミスの Location 1 ネットワークは、どちらも ExpressRoute 1 によって Azure Region 2 のリモート VNet に接続されています。

## <a name="expressroute-and-site-to-site-vpn-connectivity-in-tandem"></a>ExpressRoute とサイト間 VPN 接続を組み合わせる

###  <a name="site-to-site-vpn-over-expressroute"></a>ExpressRoute 上のサイト間 VPN

ExpressRoute Microsoft ピアリングを使用してサイト間 VPN を構成することにより、オンプレミス ネットワークと Azure VNet との間でプライベートにデータを交換することができます。 この構成を使用すれば、機密性、信頼性、整合性が確保されたデータ交換を実現できます。 また、このデータ交換には、アンチリプレイ対策も講じられています。 ExpressRoute Microsoft ピアリングを使用してトンネル モードでサイト間 IPsec VPN を構成する方法の詳細については、[ExpressRoute Microsoft ピアリング上のサイト間 VPN][S2S-Over-ExR] に関するページを参照してください。 

Microsoft ピアリングを使用したサイト間 VPN 構成に伴う主な制限はスループットです。 IPsec トンネル上では、VPN ゲートウェイの容量によりスループットが制限されます。 VPN ゲートウェイのスループットは、ExpressRoute のスループットよりも低くなります。 このシナリオでは、高いセキュリティが要求されるトラフィックには IPsec トンネルを使用し、それ以外のすべてのトラフィックにはプライベート ピアリングを使用することで、ExpressRoute の帯域幅使用率を最適化することができます。

### <a name="site-to-site-vpn-as-a-secure-failover-path-for-expressroute"></a>ExpressRoute の安全なフェールオーバー パスとしてのサイト間 VPN

ExpressRoute は、高可用性を確保する冗長回線ペアとしての役割を果たします。 異なる Azure リージョンで geo 冗長 ExpressRoute 接続を構成することができます。 また、ここで紹介したテスト セットアップのように、任意の Azure リージョン内でサイト間 VPN を使用して、ExpressRoute 接続のフェールオーバー パスを作成することができます。 ExpressRoute とサイト間 VPN の両方に同じプレフィックスがアドバタイズされた場合、Azure では ExpressRoute が優先されます。 ExpressRoute とサイト間 VPN との間で非対称なルーティングを回避するために、オンプレミス ネットワーク側も、ExpressRoute をサイト間 VPN 接続よりも優先して使用するように構成する必要があります。

ExpressRoute とサイト間 VPN が共存する接続を構成する方法の詳細については、[ExpressRoute とサイト間の共存][ExR-S2S-CoEx]に関するページを参照してください。

## <a name="extend-back-end-connectivity-to-spoke-vnets-and-branch-locations"></a>バックエンド接続をスポーク VNet とブランチ ロケーションに拡張する

### <a name="spoke-vnet-connectivity-by-using-vnet-peering"></a>VNet ピアリングを使用したスポーク VNet 接続

ハブ アンド スポークの VNet アーキテクチャは、広く利用されています。 ハブは、Azure 内の VNet であり、スポーク VNet とオンプレミス ネットワークをつなぐ接続の中央拠点として機能します。 スポークはハブとピアリングする VNet であり、ワークロードの分離に利用できます。 オンプレミスのデータセンターとハブとの間のトラフィックは、ExpressRoute または VPN 接続を経由して送信されます。 このアーキテクチャの詳細については、「[Azure にハブスポーク ネットワーク トポロジを実装する][Hub-n-Spoke]」を参照してください。

スポーク VNet は、同じリージョン内の VNet ピアリングを通じてハブ VNet ゲートウェイ (VPN ゲートウェイと ExpressRoute ゲートウェイの両方) を使用し、リモート ネットワークと通信を行うことができます。

### <a name="branch-vnet-connectivity-by-using-site-to-site-vpn"></a>サイト間 VPN を使用したブランチ VNet 接続

ブランチ VNet (異なるリージョンに存在) とオンプレミス ネットワークとをハブ VNet 経由で互いに通信させたいとしましょう。 この構成に対する Azure のネイティブ ソリューションは、VPN を使用したサイト間 VPN 接続です。 その他には、ネットワーク仮想アプライアンス (NVA) をハブでのルーティングに使用する方法があります。

詳細については、「[VPN ゲートウェイとは][VPN]」および[高可用性 NVA のデプロイ][Deploy-NVA]に関するページを参照してください。

## <a name="next-steps"></a>次の手順

テスト トポロジの[構成の詳細][Configuration]について学習する。

テスト セットアップの[コントロール プレーン分析][Control-Analysis]と、トポロジ内のさまざまな VNet や VLAN のビューについて学習する。

テスト セットアップの[データ プレーン分析][Data-Analysis]および Azure ネットワーク監視機能のビューについて学習する。

次のテーマについては、「[ExpressRoute の FAQ][ExR-FAQ]」を参照してください。
-   ExpressRoute ゲートウェイに接続できる ExpressRoute 回線の数について学習する。
-   ExpressRoute 回線に接続できる ExpressRoute ゲートウェイの数について学習する。
-   ExpressRoute のスケールの上限に関するその他の情報について学習する。


<!--Image References-->
[1]: ./media/backend-interoperability/TestSetup.png "テスト トポロジの図"

<!--Link References-->
[ExpressRoute]: https://docs.microsoft.com/azure/expressroute/expressroute-introduction
[VPN]: https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways
[VNet]: https://docs.microsoft.com/azure/virtual-network/tutorial-connect-virtual-networks-portal
[Configuration]: https://docs.microsoft.com/azure/connectivty-interoperability-configuration
[Control-Analysis]:https://docs.microsoft.com/azure/connectivty-interoperability-control-plane
[Data-Analysis]: https://docs.microsoft.com/azure/connectivty-interoperability-data-plane
[ExR-FAQ]: https://docs.microsoft.com/azure/expressroute/expressroute-faqs
[S2S-Over-ExR]: https://docs.microsoft.com/azure/expressroute/site-to-site-vpn-over-microsoft-peering
[ExR-S2S-CoEx]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-coexist-resource-manager
[Hub-n-Spoke]: https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke
[Deploy-NVA]: https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/nva-ha


