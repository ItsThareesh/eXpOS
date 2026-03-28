#!/bin/bash

# Configuration

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

BASE_DIR="$HOME/myexpos"
SPL_DIR="$BASE_DIR/spl"

# Argument Validation

if [ "$#" -lt 1 ] || [ "$#" -gt 2 ]; then
    echo -e "${YELLOW}Usage:${NC} compile_spl <stage_folder> [file_name_without_extension]"
    exit 2
fi

STAGE="$1"
FILE_FILTER="$2"
TARGET_DIR="$SPL_DIR/spl_progs/$STAGE"

if [ ! -d "$TARGET_DIR" ]; then
    echo -e "${RED}Error:${NC} Stage '$STAGE' does not exist."
    exit 2
fi

if [ -n "$FILE_FILTER" ] && [ ! -f "$TARGET_DIR/$FILE_FILTER.spl" ]; then
    echo -e "${RED}Error:${NC} File '$FILE_FILTER.spl' not found in stage '$STAGE'."
    exit 2
fi

cd "$SPL_DIR" || exit 2

# Build Mode Display

if [ -n "$FILE_FILTER" ]; then
    echo -e "${BLUE}Build Mode:${NC} Single file"
    echo -e "${YELLOW}Target:${NC} $FILE_FILTER.spl"
else
    echo -e "${BLUE}Build Mode:${NC} Full stage"
    echo -e "${YELLOW}Stage:${NC} $STAGE"
fi

echo "------------------------------------------------------------"

# Compilation

total=0
success=0
failed=0
failed_files=()

shopt -s nullglob

for file in "$TARGET_DIR"/*.spl; do
    fname=$(basename "$file" .spl)

    if [ -n "$FILE_FILTER" ] && [ "$FILE_FILTER" != "$fname" ]; then
        continue
    fi

    total=$((total + 1))

    printf "%-45s" "Compiling: $fname.spl"

    output=$(./spl "$file" 2>&1)

    if [ -n "$output" ]; then
        echo -e "[ ${RED}FAILED${NC} ]"
        # Indent the error message for better readability
        echo "$output" | sed 's/^/   /'
        failed=$((failed + 1))
        failed_files+=("$fname.spl")
    else
        echo -e "[ ${GREEN}OK${NC} ]"
        success=$((success + 1))
    fi
done

echo "------------------------------------------------------------"

# Final Report

if [ "$total" -eq 0 ]; then
    echo -e "${YELLOW}No .spl files found to compile.${NC}"
    exit 1
fi

echo -e "${BLUE}Summary:${NC}"
echo -e " - Total   : $total"
echo -e " - Success : ${GREEN}$success${NC}"
echo -e " - Failed  : ${RED}$failed${NC}"

if [ "$failed" -ne 0 ]; then
    echo
    echo -e "${RED}Failed Files:${NC}"
    for f in "${failed_files[@]}"; do
        echo -e " - $f"
    done
    exit 1
fi

echo
echo -e "${GREEN}Build completed successfully.${NC}"
exit 0