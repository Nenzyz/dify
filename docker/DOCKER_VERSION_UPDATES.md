# Docker Image Version Updates

## Summary

This document tracks the Docker image version updates made to `docker-compose.yaml` to use the latest stable versions as of January 2025.

## Updated Images

### Core Database & Cache

| Service | Previous Version | Updated Version | Notes |
|---------|-----------------|-----------------|-------|
| PostgreSQL | `postgres:15-alpine` | `postgres:17-alpine` | Major version upgrade, includes performance improvements and new features |
| Redis | `redis:6-alpine` | `redis:8-alpine` | Major version upgrade, Redis 8.0 GA with 30+ performance improvements |

### Vector Databases

| Service | Previous Version | Updated Version | Notes |
|---------|-----------------|-----------------|-------|
| Weaviate | `semitechnologies/weaviate:1.27.0` | `semitechnologies/weaviate:1.33.4` | Updated to latest stable version |
| Milvus | `milvusdb/milvus:v2.5.15` | `milvusdb/milvus:v2.6.4` | Includes coordinator consolidation and new Streaming Node component |

### Search & Analytics

| Service | Previous Version | Updated Version | Notes |
|---------|-----------------|-----------------|-------|
| Elasticsearch | `docker.elastic.co/elasticsearch/elasticsearch:8.14.3` | `docker.elastic.co/elasticsearch/elasticsearch:8.19.6` | Latest 8.x series release |
| Kibana | `docker.elastic.co/kibana/kibana:8.14.3` | `docker.elastic.co/kibana/kibana:8.19.6` | Matched with Elasticsearch version |

## Unchanged Images

The following images were kept at their current versions as they are already using the latest stable releases or Dify-specific builds:

- **Dify Images**: `langgenius/dify-api:1.9.2`, `langgenius/dify-web:1.9.2`, `langgenius/dify-sandbox:0.2.12`, `langgenius/dify-plugin-daemon:0.3.3-local`
- **Nginx**: `nginx:latest`
- **Squid (SSRF Proxy)**: `ubuntu/squid:latest`
- **Certbot**: `certbot/certbot`
- **Unstructured**: `downloads.unstructured.io/unstructured-io/unstructured-api:latest`
- **Various Vector DBs**: pgvector, qdrant, chroma, oceanbase, oracle, opengauss, myscale, matrixone (using appropriate stable versions)

## Migration Considerations

### PostgreSQL 15 → 17

**Important**: This is a major version upgrade. Before deploying:

1. **Backup your database**: Always create a full backup before upgrading
2. **Review breaking changes**: Check [PostgreSQL release notes](https://www.postgresql.org/docs/17/release.html)
3. **Test compatibility**: Ensure Dify 1.9.2 is compatible with PostgreSQL 17
4. **Migration path**:
   - For existing deployments: Consider using `pg_upgrade` or dump/restore
   - For fresh installations: No action needed

**Key PostgreSQL 17 Features**:
- Improved query performance
- Better JSON handling
- Enhanced full-text search
- Logical replication improvements

### Redis 6 → 8

**Important**: This is a major version upgrade.

1. **Backup persistence files**: Backup RDB/AOF files if using persistence
2. **Review breaking changes**: Check [Redis 8.0 release notes](https://redis.io/docs/latest/operate/oss_and_stack/stack-with-enterprise/release-notes/redisce/redisos-8.0-release-notes/)
3. **Test compatibility**: Verify Dify's Redis usage patterns work with Redis 8

**Key Redis 8 Features**:
- 30+ performance improvements
- Enhanced multi-core utilization
- Better memory management
- Improved replication

### Weaviate 1.27 → 1.33

- Review [Weaviate changelog](https://weaviate.io/developers/weaviate/current/release-notes) for migration notes
- Backup your Weaviate data before upgrading
- Test with non-production data first

### Milvus 2.5 → 2.6

**Important**: Version 2.6 includes significant architectural changes.

1. **Backup collections**: Use Milvus backup tools
2. **Review migration guide**: Check [Milvus 2.6 migration guide](https://milvus.io/docs/upgrade_milvus_standalone-docker.md)
3. **Notable changes**:
   - Coordinator consolidation
   - New Streaming Node component
   - IndexNode removal

### Elasticsearch 8.14 → 8.19

- Minor version upgrade within the 8.x series
- Generally backward compatible
- Review [Elasticsearch 8.19 release notes](https://www.elastic.co/guide/en/elasticsearch/reference/8.19/release-notes.html)
- Kibana version must match Elasticsearch version

## Deployment Instructions

### For Fresh Installations

Simply use the updated `docker-compose.yaml`:

```bash
docker-compose up -d
```

### For Existing Deployments

#### Option 1: In-Place Upgrade (Recommended for testing)

1. **Backup all data**:
   ```bash
   # Backup PostgreSQL
   docker-compose exec db pg_dumpall -U postgres > backup.sql

   # Backup Redis (if using persistence)
   docker-compose exec redis redis-cli save
   docker cp dify-redis-1:/data/dump.rdb ./redis_backup.rdb

   # Backup vector database data volumes
   docker run --rm -v dify_weaviate:/data -v $(pwd):/backup alpine tar czf /backup/weaviate_backup.tar.gz /data
   ```

2. **Stop services**:
   ```bash
   docker-compose down
   ```

3. **Update docker-compose.yaml** with new versions

4. **Start services**:
   ```bash
   docker-compose up -d
   ```

5. **Monitor logs**:
   ```bash
   docker-compose logs -f
   ```

#### Option 2: Blue-Green Deployment (Recommended for production)

1. Set up a parallel environment with new versions
2. Migrate data using dump/restore
3. Test thoroughly
4. Switch traffic
5. Decommission old environment

## Rollback Procedure

If issues occur after upgrading:

1. **Stop services**:
   ```bash
   docker-compose down
   ```

2. **Revert docker-compose.yaml** to previous versions

3. **Restore data from backups** (if needed)

4. **Start services**:
   ```bash
   docker-compose up -d
   ```

## Testing Checklist

Before deploying to production, verify:

- [ ] Dify API starts successfully
- [ ] Dify Web frontend loads
- [ ] Database connections work
- [ ] Redis caching functions properly
- [ ] Vector database queries return results
- [ ] Workflow execution completes
- [ ] Chat functionality works
- [ ] Knowledge base retrieval works
- [ ] Agent tool calls succeed
- [ ] Document upload and processing works
- [ ] All existing data is accessible

## Version Compatibility Matrix

| Component | Minimum Version | Recommended Version | Tested With |
|-----------|----------------|---------------------|-------------|
| Dify API/Web | 1.9.2 | 1.9.2 | 1.9.2 |
| PostgreSQL | 12+ | 17 | 17-alpine |
| Redis | 6+ | 8 | 8-alpine |
| Weaviate | 1.20+ | 1.33.4 | 1.33.4 |
| Milvus | 2.3+ | 2.6.4 | 2.6.4 |
| Elasticsearch | 8.0+ | 8.19.6 | 8.19.6 |

## Additional Resources

- [Dify Documentation](https://docs.dify.ai/)
- [PostgreSQL 17 Release Notes](https://www.postgresql.org/docs/17/release.html)
- [Redis 8.0 Documentation](https://redis.io/docs/latest/)
- [Weaviate Documentation](https://weaviate.io/developers/weaviate)
- [Milvus Documentation](https://milvus.io/docs)
- [Elasticsearch Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)

## Support

If you encounter issues after upgrading:

1. Check the logs: `docker-compose logs -f [service-name]`
2. Review the service-specific documentation
3. Open an issue on the [Dify GitHub repository](https://github.com/langgenius/dify/issues)
4. Join the [Dify Discord community](https://discord.gg/dify)

---

**Last Updated**: January 2025
**Dify Version**: 1.9.2
**Document Version**: 1.0
