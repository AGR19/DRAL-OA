# DRAL-OA Matcher - Setup and Usage Guide

## FIRST: Reassemble the ZIP File

Before starting, you need to reassemble the split ZIP file:

```bash
./reassemble.sh
```

This will combine `DRAL_OA_part_aa` and `DRAL_OA_part_ab` into `DRAL_OA.zip`, then extract it.

---

This guide provides step-by-step instructions to build, deploy, and test the DRAL-OA (Deep Reinforcement Alignment Learning - Ontology Alignment) matcher using Docker.

## Prerequisites

- Java 8 or higher
- Maven 3.6+
- Docker
- curl (for testing)

## Project Structure

```
plain DRAL_OA/
├── melt-master/
│   └── examples/
│       └── externalPythonMatcherWeb/
│           ├── Dockerfile
│           ├── pom.xml
│           ├── src/
│           ├── oaei-resources/
│           └── target/ (generated)
├── human.owl
├── mouse.owl
├── reference.rdf
└── README.md (this file)
```

## Step 1: Build the Maven Project

Navigate to the externalPythonMatcherWeb directory and build the project:

```bash
cd melt-master/examples/externalPythonMatcherWeb
mvn clean compile package -Dmaven.test.skip=true
```

**Note:** We skip tests due to Lombok compatibility issues with newer Java versions.

## Step 2: Copy Dependencies

Create the lib directory with all required dependencies:

```bash
mvn dependency:copy-dependencies -DoutputDirectory=target/lib
```

This automatically creates the `target/lib/` directory and copies all Maven dependencies needed for the Docker build.

## Step 3: Build Docker Image

Build the Docker image from the Dockerfile:

```bash
docker build -t dral-oa-matcher .
```

**Build time:** ~3-4 minutes (depending on your system and network speed)

## Step 4: Run Docker Container

Start the container in detached mode:

```bash
docker run -d -p 8080:8080 --name dral-oa-container dral-oa-matcher
```

Verify the container is running:

```bash
docker ps
```

You should see output similar to:
```
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                    NAMES
4f32ea4ba102   dral-oa-matcher   "java -cp dral-oa-ma…"   8 seconds ago   Up 7 seconds   0.0.0.0:8080->8080/tcp   dral-oa-container
```

## Step 5: Test the Service

Wait for the service to fully start (about 10-15 seconds), then test:

```bash
curl -I http://localhost:8080
```

Expected response: `HTTP/1.1 404 Not Found` (this is normal for the root path)

## Step 6: Extract the DRAL_OA Files

First, reassemble and extract the DRAL_OA files:

```bash
# Reassemble the split ZIP file
./reassemble.sh

# Extract the contents
unzip DRAL_OA.zip
```

## Step 7: Run Ontology Matching MAKE SURE YOU HAVE human.owl , mouse.owl and reference.rdf in the folder


Navigate to the root directory containing your ontology files and run the matching:

```bash
cd /path/to/plain\ DRAL_OA
curl -F 'source=@human.owl' -F 'target=@mouse.owl' -F 'inputAlignment=@reference.rdf' http://localhost:8080/match -o result.rdf
```

**Processing time:** ~15-20 minutes (depending on ontology size and system performance)

## Repository Files

This repository contains:
- `DRAL_OA_part_aa` and `DRAL_OA_part_ab` - Split ZIP file parts (use `./reassemble.sh` to combine)
- `reassemble.sh` - Script to reassemble the split ZIP file
- `README.md` - This setup guide

## Expected Files After Extraction

After running `./reassemble.sh` and `unzip DRAL_OA.zip`, ensure you have these files:
- `human.owl` - Source ontology
- `mouse.owl` - Target ontology  
- `reference.rdf` - Input alignment/reference mappings

## Output

The matching process will generate:
- `result.rdf` - Final alignment results in RDF format

## Troubleshooting

### Container Issues
```bash
# Check container logs
docker logs dral-oa-container

# Stop container
docker stop dral-oa-container

# Remove container
docker rm dral-oa-container

# Remove image
docker rmi dral-oa-matcher
```

### Build Issues
```bash
# Clean and rebuild
mvn clean
mvn compile package -Dmaven.test.skip=true
mvn dependency:copy-dependencies -DoutputDirectory=target/lib
```

### Port Issues
If port 8080 is already in use, change the port mapping:
```bash
docker run -d -p 8081:8080 --name dral-oa-container dral-oa-matcher
# Then use http://localhost:8081/match in your curl command
```

## What's Included

This setup includes:

1. **Dependency Management**: All Maven dependencies are automatically copied to the lib directory
2. **Docker Build**: Dockerfile works correctly with the proper lib directory structure
3. **Complete Pipeline**: Full DRAL-OA ontology matching pipeline with GNN processing

## Architecture

The DRAL-OA matcher processes ontologies through several stages:
1. **OWL to JSON conversion** - Converts input ontologies to JSON format
2. **Label processing** - Cleans and processes entity labels
3. **Word embedding generation** - Creates FastText embeddings
4. **Entity vector creation** - Generates entity representations
5. **Graph analysis** - Analyzes ontology structure
6. **GNN processing** - Applies Graph Neural Network for similarity computation
7. **Classification** - Uses BERT-based classifier for final alignment decisions

## Performance Notes

- **Memory**: Requires at least 4GB RAM for processing large ontologies
- **Processing Time**: Varies based on ontology size (15-30 minutes for anatomy track)
- **Storage**: Temporary files are created during processing and cleaned up automatically

## Support

For issues or questions:
1. Check Docker container logs: `docker logs dral-oa-container`
2. Verify all input files are present and properly formatted
3. Ensure sufficient system resources (RAM/disk space)
4. Check that port 8080 is not blocked by firewall

YOU WILL SEE SOME ISSUES/ERRORS IN DOCKER LOGS, PLEASE IGNORE.

---

**Last Updated:** August 2024  
**Version:** 1.0  
**Compatibility:** MELT 3.0, Python 3.9, Docker
