# Data Quality Assessment

## Dataset Overview

- Rows: 10,324
- Columns: 35

## Initial Findings

### Primary Key

ID

- 10,324 records
- 10,324 distinct
- 10,324 unique

Conclusion

No duplicate Shipment IDs were found.

---

### Character Encoding

One country value contained corrupted UTF encoding.

Corrected:

CÃ... → Côte d'Ivoire

---

### Initial Quality

- No parsing errors
- No duplicated IDs