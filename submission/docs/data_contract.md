What schema do you expect?

The current schema works, but it could be further normalized if needed

```bash
model vehicle {
  id            Int      @id @default(autoincrement())
  vehicle_name     String      @unique
  createdAt     DateTime @default(now())

  telemetry             vehicle_telemetry[]
  can_signals           can_signal[]
  events                vehicle_event[]
  scores                driver_score[]
  unsafe_events         unsafe_event[]
  driver_vehicle        driver_vehicle[]
}


model vehicle_telemetry {
    id              Int      @id @default(autoincrement())
    vehicle_id      Int
    source          String?
    timestamp       DateTime

    lat             Float?
    long            Float?
    speed_kmh       Float?
    heading_deg     Float?
    engine_on       Boolean?
    odometer_km     Float?
    driver_id       Int

    vehicle vehicle @relation(fields: [vehicle_id], references: [id])
    driver drivers @relation(fields: [driver_id], references: [id])

    unsafe_event       unsafe_event[]

    @@index([vehicle_id, timestamp])
    @@index([timestamp])
}

model drivers {
  id            Int      @id @default(autoincrement())
  driver_name   String   @unique
  createdAt     DateTime @default(now())


  vehicle_telemetry     vehicle_telemetry[]
  driver_vehicle        driver_vehicle[]
  unsafe_event          unsafe_event[]

}

model driver_vehicle {
  id                Int @id @default(autoincrement())
  vehicle_id        Int
  driver_id         Int

  start_at          DateTime?
  end_at            DateTime?

  vehicle vehicle @relation(fields: [vehicle_id], references: [id])
  driver drivers @relation(fields: [driver_id], references: [id])

  @@index([vehicle_id])
  @@index([driver_id])
  @@unique([vehicle_id, driver_id])
}

model can_signal {
    id          Int      @id @default(autoincrement())
    vehicle_id    Int
    timestamp    DateTime
    signal_name   String
    signal_value  Float
    signal_unit   String?

    vehicle vehicle @relation(fields: [vehicle_id], references: [id])

    @@index([vehicle_id, timestamp, signal_name])
    @@index([vehicle_id, signal_name, timestamp])
    @@index([timestamp])
}


model vehicle_event {
    id          Int      @id @default(autoincrement())
    vehicle_id         Int
    event_id         Int
    timestamp         DateTime

    lat               Float?
    long               Float?
    speed_kmh          Float?
    fuel_input_liters   Float?
    total_distance_km   Float?
    battery_voltage    Float?

    vehicle vehicle @relation(fields: [vehicle_id], references: [id])
    event event @relation(fields: [event_id], references: [id])

    @@index([vehicle_id, timestamp])
    @@index([event_id])
}


model unsafe_event {
    id         Int      @id @default(autoincrement())
    vehicle_id  Int
    driver_id   Int?
    event_id    Int
    severity    Severity        // low, medium, high
    timestamp   DateTime
    value       Float?        // speed / acceleration
    threshold   Float?    // ค่า limit
    telemetry_id Int?

    vehicle vehicle @relation(fields: [vehicle_id], references: [id])
    event event @relation(fields: [event_id], references: [id])
    driver drivers? @relation(fields: [driver_id], references: [id])
    vehicle_telemetry vehicle_telemetry? @relation(fields: [telemetry_id], references: [id])

    @@index([vehicle_id, timestamp])
    @@index([event_id])
}

model event {
    id         Int      @id @default(autoincrement())
    event_type  Int   // speeding, harsh_braking, harsh_acceleration
    event_name  String

    unsafe_event       unsafe_event[]
    vehicle_event       vehicle_event[]

    @@index([event_type])
}

model driver_score {
    id         Int      @id @default(autoincrement())
    vehicle_id       Int
    score_date       DateTime

    final_score      Float
    speeding_score   Float
    braking_score    Float
    accel_score      Float
    idle_score       Float
    night_score      Float

    riskClass       String

    vehicle vehicle @relation(fields: [vehicle_id], references: [id])

    @@index([vehicle_id, score_date])
    @@index([score_date])
}


```

What quality guarantees do you need? (freshness, completeness, null handling)

- I require strong data quality guarantees including low latency (freshness), high completeness, strict null handling, and validation rules.
  I also handle late and duplicate data through idempotency and monitoring to ensure reliable downstream processing.

How would you handle schema changes from upstream?

- Use schema versioning
  - Upstream send the version for each time
- Maintain backward compatibility
  - Ignore unknown fields
  - New version when breaking change
- Add a validation layer like schema
  - validate payload follow by version
  - Reject wrong data
- Transform schema from upstream to internal schema

What happens when upstream data is late or missing?

- Late data is accepted but flagged and may trigger recomputation, while missing data is handled gracefully using partial responses, retries, and monitoring to ensure system reliability
  Step by
- Handle late data by flag field with late status
- Check timestamp and mark something like 'Late data' to dashboard
- Retry mechanism by event queue like kafka
