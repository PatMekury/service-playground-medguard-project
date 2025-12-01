# MedGuard - Mediterranean Fisheries Management System

## Overview

MedGuard is an ecosystem-based decision support system for sustainable Mediterranean fisheries management. It provides real-time monitoring and assessment tools for fisheries managers, conservation agencies, and aquaculture operators.

## Features

- **Sustainability Risk Assessment**: Identify areas at risk of overfishing
- **Critical Habitat Mapping**: Locate spawning grounds and nursery areas
- **Larval Connectivity Analysis**: Track fish larvae dispersal corridors
- **Ecosystem Health Monitoring**: Comprehensive health indicators
- **Daily Simulation**: View conditions from January 2025 to December 2026

## Requirements

- Kubernetes cluster with Helm 3+
- Docker image built and pushed to registry
- Minimum resources: 2GB RAM, 1 CPU core

## Installation

This chart is designed to run on EDITO Model Lab infrastructure.

### Via EDITO Datalab

1. Go to EDITO Datalab: https://datalab.dive.edito.eu
2. Search for "MedGuard" in Service Catalog
3. Click "Launch"
4. Configure resources (recommended: 4GB RAM, 2 CPU)
5. Click "Create Service"

### Configuration Options

- **CPU**: 0.5, 1, or 2 cores
- **Memory**: 2Gi, 4Gi, or 8Gi
- **S3 Storage**: Optional data storage access

## Usage

Once deployed, access the dashboard through the provided URL. The interface provides:

1. **Date Selection**: Choose dates from 01/01/2025 to 31/12/2026
2. **Layer Selection**: Switch between Risk, Habitat, Connectivity, and Health views
3. **Interactive Map**: Zoom, pan, and explore Mediterranean regions
4. **Analytics**: View trends, spatial distributions, and statistics

## Data Sources

- Copernicus Marine Service
- FAO Mediterranean Fisheries Statistics
- Protected Planet MPA Database
- EDITO Model Lab Data Lake

## Support

- **Email**: patrickobumselu@gmail.com
- **Repository**: https://gitlab.com/Patmekury/medguard
- **EDITO Forum**: https://forum.dive.edito.eu/

## License

This project supports UN Sustainable Development Goal 14.4: Ending overfishing and restoring fish stocks.

## Version History

- **1.0.0** (2025): Initial release with daily simulation capability
