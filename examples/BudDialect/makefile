#!/bin/bash
BUDDY_OPT := ../../build/bin/buddy-opt

bud-array-attr-lower:
	@${BUDDY_OPT} ./TestArrayAttr.mlir \
		--lower-bud -o ./log.mlir

bud-constant-lower:
	@${BUDDY_OPT} ./TestConstant.mlir \
		--lower-bud -o ./log.mlir

bud-print-lower:
	@${BUDDY_OPT} ./TestPrint.mlir \
		--lower-bud -o ./log.mlir

bud-str-attr-lower:
	@${BUDDY_OPT} ./TestStrAttr.mlir \
		--lower-bud -o ./log.mlir
