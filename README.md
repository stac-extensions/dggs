# DGGS Extension Specification

- **Title:** DGGS
- **Identifier:** <https://stac-extensions.github.io/dggs/v0.1.0/schema.json>
- **Field Name Prefix:** dggs
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @m-mohr

This document explains the DGGS Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.

A Discrete Global Grid System (DGGS) partitions the globe into a hierarchy of zones at successive refinement levels.
Data that is organized in a DGGS is referenced by zone identifiers instead of coordinates in a
coordinate reference system (CRS).
This extension describes which Discrete Global Grid Reference System (DGGRS) the data is organized in,
at which refinement level(s) the data is available, and optionally which zone a STAC entity covers.
This enables searching for DGGS data and gives clients the information they need to load and process the data.

The terminology follows [OGC API - DGGS](https://docs.ogc.org/is/21-038r1/21-038r1.html) and
[OGC Topic 21 / ISO 19170-1](https://docs.ogc.org/as/20-040r3/20-040r3.html),
e.g. the term *zone* is used instead of *cell*.
The fields are mostly aligned with the [DGGS convention for Zarr](https://github.com/zarr-conventions/dggs)
so that metadata can be mapped between STAC and the data files.

- Examples:
  - [Item example](examples/item.json): A STAC Item that covers a single HEALPix zone,
    with one asset per refinement level
  - [Collection example](examples/collection.json): A STAC Collection that contains Items as in the Item example
  - [Data cube example](examples/datacube.json): A STAC Item with a DGGS data cube based on HEALPix
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links
- [x] [Data cube dimensions](#data-cubes) of type `dggs`

| Field Name            | Type    | Description                                                                                                          |
| --------------------- | ------- | -------------------------------------------------------------------------------------------------------------------- |
| dggs:dggrs            | string  | Name of the [DGGRS](#dggsdggrs) that the data is organized in.                                                       |
| dggs:refinement_level | integer | The [refinement level](#dggsrefinement_level) of the zones that the data values are provided for.                    |
| dggs:indexing_scheme  | string  | The [indexing scheme](#dggsindexing_scheme) of the zone identifiers, if it differs from the definition of the DGGRS. |
| dggs:zone_id          | string  | Identifier of the [zone](#dggszone_id-and-dggszone_level) that the Item or Asset covers spatially.                   |
| dggs:zone_level       | integer | The refinement level of the zone in `dggs:zone_id`. RECOMMENDED if `dggs:zone_id` is given.                          |

### Additional Field Information

#### dggs:dggrs

The name of the Discrete Global Grid Reference System (DGGRS), i.e. the grid hierarchy together with
its zone identifier reference system and sub-zone ordering.

For the DGGRS in the table below, the name from the table MUST be used (case-sensitive).
Other DGGRS MAY use other names, which SHOULD NOT conflict with the names in the table.
In this case, a link to the definition of the DGGRS SHOULD be provided.
Please propose commonly used DGGRS for inclusion in the table by opening an issue or PR.

For the DGGRS registered by OGC, the names are the identifiers used in the [OGC DGGRS Register](https://www.opengis.net/def/dggrs).

| Name             | Registered by OGC | Description                                                                             |
| ---------------- | ----------------- | --------------------------------------------------------------------------------------- |
| ISEA3H           | Yes               | Icosahedral Snyder Equal Area, hexagonal zones, refinement ratio 3                      |
| ISEA7H           | Yes               | Icosahedral Snyder Equal Area, hexagonal zones, refinement ratio 7                      |
| ISEA9R           | Yes               | Icosahedral Snyder Equal Area, rhombic zones, refinement ratio 9                        |
| IVEA3H           | Yes               | Icosahedral Vertex-oriented Equal Area, hexagonal zones, refinement ratio 3             |
| IVEA7H           | Yes               | Icosahedral Vertex-oriented Equal Area, hexagonal zones, refinement ratio 7             |
| IVEA9R           | Yes               | Icosahedral Vertex-oriented Equal Area, rhombic zones, refinement ratio 9               |
| HEALPix          | Yes               | Hierarchical Equal Area isoLatitude Pixelization, refinement ratio 4                    |
| rHEALPix         | Yes               | Rearranged HEALPix, refinement ratio 9                                                  |
| GNOSISGlobalGrid | Yes               | GNOSIS Global Grid, based on the 2D Tile Matrix Set of the same name                    |
| H3               | No                | [H3](https://h3geo.org), mostly hexagonal zones, refinement ratio 7                     |
| S2               | No                | [S2 Geometry](https://s2geometry.io), quadrilateral zones on a cube, refinement ratio 4 |
| A5               | No                | [A5](https://a5geo.org), pentagonal zones on a dodecahedron                             |
| IGEO7            | No                | ISEA7H with Z7 zone identifiers                                                         |

It is RECOMMENDED to provide a link with the relation type
`https://www.opengis.net/def/rel/ogc/1.0/dggrs-definition` that points to a definition of the DGGRS
(see [Relation types](#relation-types)).

#### dggs:refinement_level

The refinement level of the zones that the data values are provided for, i.e. the resolution of the data.
In other specifications, it is also called resolution, order or depth.
Level 0 is the level with the fewest and largest zones.
The refinement levels are specific to the DGGRS and can't be compared across DGGRS.

If the data is available in multiple refinement levels, it is RECOMMENDED to provide a separate Asset per refinement level,
each with its own `dggs:refinement_level`.
If a single Asset contains multiple refinement levels (e.g. a multiscale Zarr), `dggs:refinement_level`
is the finest refinement level contained in the Asset.

#### dggs:indexing_scheme

The indexing scheme that the zone identifiers follow, if it differs from the zone identifier reference system
defined for the DGGRS.
If this field is not provided, the zone identifiers follow the definition of the DGGRS.

Some grids are commonly used with indexing schemes that differ from the DGGRS definition.
For example, the OGC definition of HEALPix uses textual zone identifiers such as `C1-A`,
while most HEALPix data uses integer zone identifiers in the `nested` or `ring` scheme.
The same integer identifies different zones in these schemes,
so the indexing scheme is needed to interpret the zone identifiers.

The values SHOULD follow the [DGGS-specific parameters](https://github.com/zarr-conventions/dggs#dggs-specific-parameters)
of the DGGS convention for Zarr, e.g. `nested`, `ring`, `zuniq` or `nuniq` for HEALPix.

#### dggs:zone_id and dggs:zone_level

If the data is split into files by zone (e.g. one Item per zone at a coarse refinement level),
`dggs:zone_id` is the identifier of the zone that the Item or Asset covers spatially.
`dggs:zone_level` is the refinement level of this zone.
It SHOULD be provided because zone identifiers don't encode their refinement level in all DGGRS,
e.g. in HEALPix with the `nested` indexing scheme.

`dggs:zone_level` describes the spatial partitioning of the files,
while `dggs:refinement_level` describes the resolution of the data values.
For example, an Item with `dggs:zone_level` 7 and `dggs:refinement_level` 13 covers a single zone at level 7
and contains the data values for all zones at level 13 within it.
`dggs:zone_level` is always smaller than or equal to `dggs:refinement_level`.

Zone identifiers MUST be provided as strings.
If the DGGRS or the indexing scheme (see `dggs:indexing_scheme`) defines textual zone identifiers, they MUST be used.
Otherwise, the decimal representation of the integer zone identifiers MUST be used.

The `geometry` and `bbox` of an Item SHOULD cover the zone and all data in the Item.
Depending on the DGGRS, zones at finer refinement levels are not necessarily nested in the coarser zones (e.g. in ISEA3H).

### Reference system and ellipsoid

The DGGRS usually determines the coordinate reference system (CRS) that it is based on, including the ellipsoid.
If a grid is used with a different CRS or ellipsoid than defined for the DGGRS (e.g. HEALPix on a sphere),
the CRS SHOULD be provided through the fields `proj:code`, `proj:wkt2` or `proj:projjson` of the
[Projection Extension](https://github.com/stac-extensions/projection).
The ellipsoid can be read from the CRS definition, e.g. the `ellipsoid` object in PROJJSON,
which is also the model for the ellipsoid in the DGGS convention for Zarr.

The other fields of the Projection Extension (e.g. `proj:shape` and `proj:transform`) describe raster grids and
SHOULD NOT be provided for DGGS data.

## Data cubes

The [Datacube Extension](https://github.com/stac-extensions/datacube) defines a dimension type `dggs`
for data cubes with a dimension of DGGS zones.
In such a dimension, the fields `reference_system` and `resolution` are equivalent to
`dggs:dggrs` and `dggs:refinement_level`.
Either or both of them MAY be provided in the dimension.
If both are provided, they MUST describe the same DGGRS and refinement level,
e.g. `reference_system` may contain the URI `https://www.opengis.net/def/dggrs/OGC/1.0/HEALPix`
while `dggs:dggrs` contains the name `HEALPix`.
The other fields of this extension (e.g. `dggs:indexing_scheme`) MAY be added to the dimension.

It is RECOMMENDED to additionally provide `dggs:dggrs` and `dggs:refinement_level`
in the Item Properties or Assets so that the data can be found by searching.

## Relation types

The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).
They are defined by OGC API - DGGS.

| Type                                                       | Description                                                                            |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| <https://www.opengis.net/def/rel/ogc/1.0/dggrs-definition> | Points to the definition of the DGGRS.                                                 |
| <https://www.opengis.net/def/rel/ogc/1.0/dggrs-list>       | Points to the list of DGGRS that the data is available in through an OGC API - DGGS.   |
| <https://www.opengis.net/def/rel/ogc/1.0/dggrs-zone-data>  | Points to the data of the zone given in `dggs:zone_id`, provided by an OGC API - DGGS. |

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid.
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on
your command line run:

```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:

```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:

```bash
npm run format-examples
```
