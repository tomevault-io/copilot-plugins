## droneengage-scriptwiki

> whenever creating a bash script and need to output test use the following output functions for logging


whenever creating a bash script and need to output test use the following output functions for logging


RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DroneEngage) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
