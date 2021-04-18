## MM285 Manifest Column Header

- 1-3: chrm, beg, end of target, length 2 for CpG, length 1 for SNP and CpH. beg is 0-based and end is 1-based like in bed files.
- 4-5: tango address for M and U
- 6: target: CpG, the target reference allele
- 7: extension base
- 8: col: color channel, green or red
- 9: probe: ID
- 10: last: last base on 3'-end of the probe sequence, for Infinium-II CpG probe it will always be C
- 11-19: mapFlag, mapChrm, mapPos, mapQ, mapCigar, mapSeq, mapNM, mapAS, mapYD
- 20: lastB: last for B-allele for Infinium-I probes
- 21-29: mapFlag.B, mapChrm.B, mapPos.B, mapQ.B, mapCigar.B, mapSeq.B, mapNM.B, mapAS.B, mapYD.B
- 30: design category string
