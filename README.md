# MPAS-Urban (HKUST)

MPAS-Urban is a HKUST-modified version of the MPAS-Atmosphere core based on the official MPAS-Model v8.2.2 release. This branch extends the Noah-MP land surface scheme with the Single-Layer Urban Canopy Model (SLUCM) for urban climate applications.
Maintained by Yeer Cao and Zhuo Liu

## Main Features

- Based on the official MPAS-Model v8.2.2 atmosphere core.
- Adds SLUCM coupling to Noah-MP.
- Computes urban surface energy balance and urban canopy parameters through SLUCM.
- Provides additional diagnostics for urban surface variables in MPAS output.
- BEP/BEM is not coupled in this branch; SLUCM is the available urban canopy option.

## Static Geographical Data

The LCZ-based static geographical dataset for MPAS-Urban is provided as:

```text
mpas_cglc_lcz.tar.gz
```

Users should extract this dataset into the directory used by:

```text
config_geog_data_path
```

To use the LCZ-based land-use data during static interpolation, set the following option in `namelist.init_atmosphere`:

```text
config_landuse_data = 'CGLC_LCZ'
```

## Runtime Tables

Additional urban runtime tables are stored in:

```text
src/core_atmosphere/physics/custom_wrf_files/
```

The current custom urban tables are:

```text
URBPARM_UZE.TBL
URBPARM_LCZ.TBL
URBPARM.TBL
```

During compilation, the script:

```text
src/core_atmosphere/physics/install_custom_wrf_files.sh
```

copies these urban tables, together with:

```text
src/core_atmosphere/physics/physics_noahmp/parameters/NoahmpTable.TBL
```

into:

```text
src/core_atmosphere/physics/physics_wrf/files/
```

The standard MPAS build then links `*TBL` and `*DATA*` files from `physics_wrf/files/` to the top-level MPAS run directory.

## Enabling SLUCM

To run MPAS-Urban with SLUCM, enable urban physics in `namelist.atmosphere`:

```text
config_urban_physics = true
```

When `config_urban_physics = true`, SLUCM is used as the urban canopy scheme. When `config_urban_physics = false`, the model falls back to the standard Noah-MP land surface configuration without urban canopy processes.

## Initialization Notes

When creating initial conditions with `init_atmosphere_model`, users should ensure that the first atmospheric model level is high enough for the urban canopy parameters used in the simulation.

In particular, avoid the warning:

```text
Warning ZR : Mean Height Table + 2 m is larger than the 1st WRF level
```

This warning indicates that the mean building height specified by the urban parameter table, plus the 2 m diagnostic height, exceeds the height of the first atmospheric level. In that case, the lowest atmospheric level is too low for the prescribed urban canopy geometry, which can affect surface-layer exchange and SLUCM coupling.

Users should therefore check the local building-height parameters in `URBPARM*.TBL` and set the vertical grid in `namelist.init_atmosphere` so that the first atmospheric level is above `ZR + 2 m` for the local urban categories used in the domain.

Users may modify `config_specified_zeta_levels` in the `&vertical_grid` namelist group of `namelist.init_atmosphere` to point to a custom vertical-coordinate file. This is useful for urban simulations where the lowest model level must be adjusted to remain above the local urban canopy height constraint.

An example vertical level file (56 levels, first level at ~76 m, model top at 30 km) is provided in:

```text
vertical_levels/urban_ZR_75.txt
```

To use it, set:

```text
config_specified_zeta_levels = 'vertical_levels/urban_ZR_75.txt'
```

## General Usage

Build and run instructions for the general MPAS framework and MPAS-Atmosphere core are unchanged from the official v8.2.2 release. Please refer to the official MPAS-Atmosphere user guide and documentation on the MPAS-Dev website for the standard workflow.

This repository documents the urban extensions on top of MPAS v8.2.2. Other model components and workflows follow the official MPAS distribution.
