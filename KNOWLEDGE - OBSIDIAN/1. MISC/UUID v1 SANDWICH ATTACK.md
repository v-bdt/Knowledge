
https://github.com/intruder-io/guidtool

1. Récupérer un UUID valide
2. l'inspecter avec GUIDTOOL
```sh
guidtool -i <uuid>
```

3. Utiliser le script Python suivant pour générer un UUID précis à partir d'une date

```python
#!/usr/bin/env python3
import argparse
import datetime
import uuid
import sys

UUID_EPOCH = datetime.datetime(1582, 10, 15, tzinfo=datetime.timezone.utc)

def generate_uuid_v1(dt, clock_seq, mac):
    delta = dt - UUID_EPOCH

    timestamp = (
        delta.days * 86400 * 10_000_000 +
        delta.seconds * 10_000_000 +
        delta.microseconds * 10
    )

    time_low = timestamp & 0xFFFFFFFF
    time_mid = (timestamp >> 32) & 0xFFFF
    time_hi = (timestamp >> 48) & 0x0FFF
    time_hi_and_version = time_hi | (1 << 12)

    clock_seq_low = clock_seq & 0xFF
    clock_seq_hi = (clock_seq >> 8) & 0x3F
    clock_seq_hi_and_reserved = clock_seq_hi | 0x80  # RFC 4122

    return uuid.UUID(fields=(
        time_low,
        time_mid,
        time_hi_and_version,
        clock_seq_hi_and_reserved,
        clock_seq_low,
        mac
    ))

def parse_datetime(value):
    try:
        # Format attendu : YYYY-MM-DD HH:MM:SS.microseconds
        dt = datetime.datetime.fromisoformat(value)
        if dt.tzinfo is None:
            dt = dt.replace(tzinfo=datetime.timezone.utc)
        return dt.astimezone(datetime.timezone.utc)
    except ValueError:
        raise argparse.ArgumentTypeError(
            "Format invalide. Utilise : YYYY-MM-DD HH:MM:SS.microseconds"
        )

def main():
    parser = argparse.ArgumentParser(
        description="Génération déterministe d’un UUID v1 (RFC 4122)"
    )

    parser.add_argument(
        "--time",
        required=True,
        type=parse_datetime,
        help="Date UTC (ex: '2026-01-10 03:10:43.636498')"
    )

    parser.add_argument(
        "--clock-seq",
        required=True,
        type=int,
        help="Clock sequence (0–16383)"
    )

    parser.add_argument(
        "--mac",
        required=True,
        help="mac/MAC (ex: 0242ac100028 ou 02:42:ac:10:00:28)"
    )

    args = parser.parse_args()

    # Validation clock sequence
    if not (0 <= args.clock_seq <= 0x3FFF):
        sys.exit("Erreur : clock-seq doit être compris entre 0 et 16383")

    # Normalisation mac
    mac_str = args.mac.replace(":", "").lower()
    if len(mac_str) != 12:
        sys.exit("Erreur : mac/MAC doit faire 48 bits (12 hex digits)")

    try:
        mac = int(mac_str, 16)
    except ValueError:
        sys.exit("Erreur : mac/MAC invalide")

    u = generate_uuid_v1(args.time, args.clock_seq, mac)
    print(u)

if __name__ == "__main__":
    main()
```

```sh
python3 uuid_gen.py --time "2026-01-10 18:46:16.855186" --clock-seq 3709 --mac 02:42:ac:10:00:28
```

4. Le uuid généré doit ressembler à quelque chose comme ça : a56d49b4-ee54-11f0-8e7d-0242ac100028

5. Tester de changer manuellement le dernier octet du **time low** (de 0 à f) 

![UUIDs versus IDs :: Feasible Commerce](https://feasiblecommerce.uraweb.com/application/files/3416/8598/2191/uuid.webp)