## blue-green-docker-deployment

> This is a Blue/Green deployment experiment using Docker Compose, HAProxy, and NGINX to achieve zero-downtime deployments.

# CLAUDE.md - Project Context

## Project Overview
This is a Blue/Green deployment experiment using Docker Compose, HAProxy, and NGINX to achieve zero-downtime deployments.

## Key Technical Details

### HAProxy Configuration
- Uses runtime API for dynamic backend switching
- Socket path: `/var/lib/haproxy/haproxy.sock`
- Stats enabled on port 8404
- Backend pools: backend_blue, backend_green

### Service Ports
- HAProxy: 80 (public), 8404 (stats)
- NGINX Blue: 8081 (internal)
- NGINX Green: 8082 (internal)

### Automation Scripts Location
All scripts are in `./scripts/` directory:
- `deploy.sh` - Main deployment orchestrator
- `switch.sh` - Backend switching logic
- `health-check.sh` - Service health validation
- `rollback.sh` - Quick rollback mechanism
- `test-zero-downtime.sh` - Zero-downtime testing

## Important Commands

### Docker Commands
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f [service-name]

# Restart a service
docker-compose restart [service-name]

# Execute commands in container
docker-compose exec haproxy sh
```

### HAProxy Runtime API Commands
```bash
# Check backend status
echo "show servers state" | docker-compose exec -T haproxy socat stdio /var/lib/haproxy/haproxy.sock

# Disable a backend
echo "disable server backend_blue/nginx-blue" | docker-compose exec -T haproxy socat stdio /var/lib/haproxy/haproxy.sock

# Enable a backend
echo "enable server backend_green/nginx-green" | docker-compose exec -T haproxy socat stdio /var/lib/haproxy/haproxy.sock

# Set server to maintenance
echo "set server backend_blue/nginx-blue state maint" | docker-compose exec -T haproxy socat stdio /var/lib/haproxy/haproxy.sock

# Set server to ready
echo "set server backend_green/nginx-green state ready" | docker-compose exec -T haproxy socat stdio /var/lib/haproxy/haproxy.sock
```

### Testing Commands
```bash
# Quick health check
curl http://localhost/health

# Check which backend is active
curl -s http://localhost/ | grep -o "Blue\|Green"

# Load test
ab -n 1000 -c 10 http://localhost/

# Continuous monitoring
watch -n 0.5 'curl -s http://localhost/ | grep -o "Blue\|Green"'
```

## File Locations

### Configuration Files
- HAProxy config: `./haproxy/haproxy.cfg`
- Docker Compose: `./docker-compose.yml`
- Environment variables: `./.env`

### NGINX Content
- Blue content: `./nginx-blue/html/`
- Green content: `./nginx-green/html/`

### State Files
- Current deployment: `./scripts/.current_deployment`
- Deployment history: `./scripts/.deployment_history`

## Deployment Workflow

1. **Pre-deployment checks**
   - Verify Docker services are running
   - Check target environment (blue/green) is healthy
   - Validate HAProxy configuration

2. **Deployment process**
   - Update inactive container
   - Wait for health checks to pass
   - Switch traffic via HAProxy runtime API
   - Verify deployment success

3. **Post-deployment**
   - Update state files
   - Log deployment event
   - Monitor error rates

## Troubleshooting

### Common Issues

#### HAProxy won't start
- Check port 80 availability: `lsof -i :80`
- Validate config: `docker-compose exec haproxy haproxy -c -f /usr/local/etc/haproxy/haproxy.cfg`

#### Backend not switching
- Check HAProxy socket: `docker-compose exec haproxy ls -la /var/lib/haproxy/haproxy.sock`
- Verify backend status: Use runtime API commands above

#### Health checks failing
- Check endpoint: `curl http://localhost:8081/health`
- Review container logs: `docker-compose logs nginx-blue`

### Debug Mode
Enable verbose logging in scripts:
```bash
DEBUG=1 ./scripts/deploy.sh green
```

## Testing Scenarios

### Zero-Downtime Test
```bash
# Terminal 1: Start continuous requests
./scripts/test-zero-downtime.sh

# Terminal 2: Perform deployment
./scripts/deploy.sh green

# Terminal 1: Check for any failures
```

### Rollback Test
```bash
# Deploy to green
./scripts/deploy.sh green

# Simulate issue and rollback
./scripts/rollback.sh

# Verify blue is active again
curl http://localhost/
```

## Performance Tuning

### HAProxy Optimization
- `maxconn`: Adjust based on expected load
- `timeout connect`: Lower for faster failure detection
- `timeout server`: Adjust based on application response time

### Health Check Tuning
- `inter`: Interval between checks (default: 2s)
- `fall`: Failures before marking down (default: 3)
- `rise`: Successes before marking up (default: 2)

## Security Notes

### Production Considerations
- Secure HAProxy stats page with authentication
- Use TLS/SSL for public endpoints
- Restrict runtime API access
- Implement rate limiting
- Add security headers

### Development Setup
- Stats page is open for easy monitoring
- Runtime API accessible without authentication
- Debug logging enabled

## Additional Resources

### HAProxy Documentation
- Runtime API: https://www.haproxy.com/documentation/hapee/latest/api/runtime-api/
- Configuration: https://www.haproxy.com/documentation/hapee/latest/configuration/

### Docker Compose
- Networking: https://docs.docker.com/compose/networking/
- Environment variables: https://docs.docker.com/compose/environment-variables/

## Contact & Support
Project repository: sendlayer-bg-deployment-with-docker-compose
Purpose: Zero-downtime deployment experimentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cemhurturk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
