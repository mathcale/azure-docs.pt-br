---
title: Geometrias GeoJSON estendidas | Mapas do Microsoft Azure
description: Neste artigo, você aprenderá sobre como o Microsoft Azure Maps estende a especificação GeoJSON para representar certas geometrias.
author: sataneja
ms.author: sataneja
ms.date: 05/17/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.openlocfilehash: 98db10f0fc7a417f39d4bb00e77af6bdea034a03
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79276394"
---
# <a name="extended-geojson-geometries"></a>Geometrias GeoJSON estendidas

O Azure Maps fornece uma lista de APIs poderosas para pesquisar dentro e ao longo de recursos geográficos. Essas APIs aderem à [especificação padrão geojson][1] de representar características geográficas.  

A [especificação GeoJSON][1] suporta apenas as seguintes geometrias:

* GeometryCollection
* LineString
* MultiLineString
* MultiPoint
* MultiPolygon
* Point
* Polygon

Algumas APIs do Azure Maps aceitam geometrias que não fazem parte da [especificação GeoJSON][1]. Por exemplo, a API [Search Inside Geometry](https://docs.microsoft.com/rest/api/maps/search/postsearchinsidegeometry) aceita Círculos e Polígonos.

Este artigo fornece uma explicação detalhada sobre como o Azure Mapas estende a [especificação GeoJSON][1] para representar determinadas geometrias.

## <a name="circle"></a>Circle

A `Circle` geometria não é suportada pela [especificação GeoJSON][1]. Usamos um `GeoJSON Point Feature` objeto para representar um círculo.

Uma `Circle` geometria `GeoJSON Feature` representada usando o objeto __deve__ conter as seguintes coordenadas e propriedades:

- Centro

    O centro do círculo é `GeoJSON Point` representado usando um objeto.

- Raio

    O círculo `radius` é representada usando as propriedades de `GeoJSON Feature`. O valor de raio está em _metros_ e deve ser do tipo `double`.

- SubType

    A geometria do círculo também deve conter a propriedade `subType`. Esta propriedade deve ser `GeoJSON Feature`uma parte das propriedades e seu valor deve ser _Circle_

#### <a name="example"></a>Exemplo

Aqui está como você vai representar `GeoJSON Feature` um círculo usando um objeto. Vamos centralizar o círculo em latitude: 47.639754 e longitude: -122.126986, e atribuí-lo um raio igual a 100 metros:

```json            
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [-122.126986, 47.639754]
    },
    "properties": {
        "subType": "Circle",
        "radius": 100
    }
}          
```

## <a name="rectangle"></a>Retângulo

A `Rectangle` geometria não é suportada pela [especificação GeoJSON][1]. Usamos um `GeoJSON Polygon Feature` objeto para representar um retângulo. A extensão retângulo é usada principalmente pelo módulo de ferramentas de desenho do Web SDK.

Uma `Rectangle` geometria `GeoJSON Polygon Feature` representada usando o objeto __deve__ conter as seguintes coordenadas e propriedades:

- Cantos

    Os cantos do retângulo são representados `GeoJSON Polygon` usando as coordenadas de um objeto. Deve haver cinco coordenadas, uma para cada canto. E uma quinta coordenada que é a mesma da primeira coordenada para fechar o anel do polígono. Presume-se que essas coordenadas se alinhem, e que o desenvolvedor possa rodá-las conforme desejado.

- SubType

    A geometria retângulo `subType` também deve conter a propriedade. Esta propriedade deve ser `GeoJSON Feature`uma parte das propriedades, e seu valor deve ser _Retângulo_

### <a name="example"></a>Exemplo

```json
{
    "type": "Feature",
    "geometry": {
        "type": "Polygon",
        "coordinates": [[[5,25],[14,25],[14,29],[5,29],[5,25]]]
    },
    "properties": {
        "subType": "Rectangle"
    }
}

```
## <a name="next-steps"></a>Próximas etapas

Saiba mais sobre os dados do GeoJSON no Azure Maps:

> [!div class="nextstepaction"]
> [Formato Geofence GeoJSON](geofence-geojson.md)

Revise o glossário de termos técnicos comuns associados ao Azure Maps e aplicativos de inteligência de localização:

> [!div class="nextstepaction"]
> [Glossário do Azure Maps](glossary.md)

[1]: https://tools.ietf.org/html/rfc7946
