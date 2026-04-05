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
