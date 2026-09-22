enum Role { ADMIN, MANAGER }

enum JobStatus { QUEUED, PROCESSING, COMPLETED, FAILED, RETRYING }

enum ProductLifecycle {
  DISCOVERED
  ANALYZING
  REVIEWING
  APPROVAL_REQUIRED
  APPROVED
  REJECTED
  PUBLISHED
  FAILED
}

enum ApprovalStatus { PENDING, APPROVED, REJECTED, EXPIRED }

model User {
  id            String         @id @default(uuid())
  email         String         @unique
  passwordHash  String
  role          Role           @default(ADMIN)
  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt
  approvals     Approval[]
  auditLogs     AuditLog[]
}

model Product {
  id             String           @id @default(uuid())
  externalId     String?          @unique
  title          String
  description    String?
  costPrice      Decimal
  sellingPrice   Decimal
  supplierId     String
  supplier       Supplier         @relation(fields: [supplierId], references: [id])
  lifecycle      ProductLifecycle @default(DISCOVERED)
  riskScore      Float?
  profitMargin   Float?
  metadata       Json?
  createdAt      DateTime         @default(now())
  updatedAt      DateTime         @updatedAt
  aiJobs         AiJob[]
  orders         Order[]
}

model Supplier {
  id            String    @id @default(uuid())
  name          String
  apiEndpoint   String?
  rating        Float?
  isTrusted     Boolean   @default(false)
  products      Product[]
  createdAt     DateTime  @default(now())
}

model AiJob {
  id           String      @id @default(uuid())
  modelName    String      // GEMINI, OPENAI, CLAUDE
  taskType     String      // IMAGE_ANALYSIS, COPYWRITING, RISK_ASSESSMENT
  status       JobStatus   @default(QUEUED)
  inputPayload Json
  outputResult Json?
  promptTokens Int         @default(0)
  completionTokens Int     @default(0)
  estimatedCost Float      @default(0.0)
  productId    String?
  product      Product?    @relation(fields: [productId], references: [id])
  createdAt    DateTime    @default(now())
  updatedAt    DateTime    @updatedAt
}

model Approval {
  id            String         @id @default(uuid())
  entityType    String         // PRODUCT, CAMPAIGN, FINANCIAL_TX
  entityId      String
  status        ApprovalStatus @default(PENDING)
  riskLevel     String         // LOW, MEDIUM, HIGH
  payloadData   Json
  reason        String
  userId        String?
  user          User?          @relation(fields: [userId], references: [id])
  decidedAt     DateTime?
  createdAt     DateTime       @default(now())
}

model FinancialTransaction {
  id            String   @id @default(uuid())
  amount        Decimal
  currency      String   @default("USD")
  type          String   // DEPOSIT, PAYOUT, SUPPLIER_PAYMENT
  status        String   // PENDING, COMPLETED, FAILED
  idempotencyKey String  @unique
  createdAt     DateTime @default(now())
}

model AuditLog {
  id        String   @id @default(uuid())
  userId    String?
  user      User?    @relation(fields: [userId], references: [id])
  action    String
  details   Json
  ipAddress String?
  createdAt DateTime @default(now())
}

model SystemEvent {
  id        String   @id @default(uuid())
  event     String
  payload   Json
  createdAt DateTime @default(now())
}

