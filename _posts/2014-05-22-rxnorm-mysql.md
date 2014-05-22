---
layout: post
title:  "MySQL Key Length Maximum meets RxNorm"
date:   2014-05-22 09:54:00
categories: mysql rxnorm
---

The [UMLS][UMLS] [RxNorm][RxNorm] distribution includes scripts for MySQL and Oracle that import and index the data.

The `Indexes_mysql_rxn.sql` file does not work out-of-the-box because it attempts to index several columns that are bigger than the allowable key size for InnoDB (and MyISAM) indexes:

    % mysql rxnorm < Indexes_mysql_rxn.sql
    ERROR 1071 (42000) at line 1: Specified key was too long; max key length is 767 bytes

The solution is simple: specify the field length in the `CREATE INDEX` statement:

    --- scripts/mysql/Indexes_mysql_rxn.sql.orig	2014-05-22 10:04:50.000000000 -0400
    +++ scripts/mysql/Indexes_mysql_rxn.sql	2014-05-22 10:04:04.000000000 -0400
    @@ -1,11 +1,11 @@
    -CREATE INDEX X_RXNCONSO_STR ON RXNCONSO(STR);
    +CREATE INDEX X_RXNCONSO_STR ON RXNCONSO(STR(255));
     CREATE INDEX X_RXNCONSO_RXCUI ON RXNCONSO(RXCUI);
     CREATE INDEX X_RXNCONSO_TTY ON RXNCONSO(TTY);
     CREATE INDEX X_RXNCONSO_CODE ON RXNCONSO(CODE);
    
     CREATE INDEX X_RXNSAT_RXCUI ON RXNSAT(RXCUI);
    -CREATE INDEX X_RXNSAT_ATV ON RXNSAT(ATV);
    -CREATE INDEX X_RXNSAT_ATN ON RXNSAT(ATN);
    +CREATE INDEX X_RXNSAT_ATV ON RXNSAT(ATV(255));
    +CREATE INDEX X_RXNSAT_ATN ON RXNSAT(ATN(255));
    
     CREATE INDEX X_RXNREL_RXCUI1 ON RXNREL(RXCUI1);
     CREATE INDEX X_RXNREL_RXCUI2 ON RXNREL(RXCUI2);

This assumes that the database is using the `utf8` character set. When MySQL determines the storage needed for a `CHAR` or `VARCHAR` column it uses the maximum possible width of a character in the active character set. For `utf8` this is 3 bytes per character[^1], so the maximum key length of 767 bytes can contain 255 `utf8` encoded characters (255 * 3 = 765).

[RxNorm]: http://www.nlm.nih.gov/research/umls/rxnorm/
[UMLS]: http://www.nlm.nih.gov/research/umls/

[^1]: `utf8` only covers the range U+0000 --- U+FFFF, while `utf8mb4` requires a maximum of 4 bytes per character because it supports the full range U+000000 -- U+10FFFF.
