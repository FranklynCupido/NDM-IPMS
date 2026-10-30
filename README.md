// Prisma Schema for Ignite IPMS Platform (Local Government Municipality)

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

enum Role {
  EMPLOYEE
  SUPERVISOR
  DMC_CHAIR
  MMC_MEMBER
  HR_ADMIN
}

enum Directorate {
  OFFICE_OF_THE_MUNICIPAL_MANAGER
  CORPORATE_COMMUNITY_SERVICES
  FINANCIAL_SERVICES
  TECHNICAL_SERVICES
}

enum ReviewQuarter {
  Q1
  Q2_MID_YEAR
  Q3
  Q4_FINAL
}

enum ModerationStatus {
  DRAFT
  SUBMITTED_TO_SUPERVISOR
  SUPERVISOR_APPROVED
  DMC_MODERATED
  MMC_APPROVED
}

model User {
  id              String                 @id @default(uuid())
  staffNumber     String                 @unique
  fullName        String
  email           String                 @unique
  role            Role                   @default(EMPLOYEE)
  directorate     Directorate
  position        String
  supervisorId    String?
  supervisor      User?                  @relation("SupervisorDirects", fields: [supervisorId], references: [id])
  subordinates    User[]                 @relation("SupervisorDirects")
  agreements      PerformanceAgreement[]
  auditLogs       AuditLog[]
  createdAt       DateTime               @default(now())
  updatedAt       DateTime               @updatedAt
}

model PerformanceAgreement {
  id                String                  @id @default(uuid())
  financialYear     String                  // e.g. "2025-2026"
  userId            String
  user              User                    @relation(fields: [userId], references: [id])
  status            ModerationStatus        @default(DRAFT)
  kpaWeightTotal    Float                   @default(80.0)
  compWeightTotal   Float                   @default(20.0)
  kpaFinalScore     Float?
  compFinalScore    Float?
  overallFinalScore Float?
  rewardEligible    Boolean                 @default(false)
  rewardPercentage  Float?
  
  kpis              KPI[]
  competencies      CompetencyScore[]
  poeDocuments      PortfolioOfEvidence[]
  moderationLogs    ModerationLog[]
  pdp               PersonalDevelopmentPlan?
  createdAt         DateTime                @default(now())
  updatedAt         DateTime                @updatedAt
}

model KPI {
  id                  String               @id @default(uuid())
  agreementId         String
  agreement           PerformanceAgreement @relation(fields: [agreementId], references: [id])
  kpaNumber           Int
  kpaName             String
  kpiDescription      String
  target              String
  evidenceRequired    String
  unitOfMeasurement   String
  weight              Float
  isNotMeasured       Boolean              @default(false)
  
  selfScoreQ2         Float?
  supervisorScoreQ2   Float?
  finalScoreQ2        Float?
  
  selfScoreQ4         Float?
  supervisorScoreQ4   Float?
  finalScoreQ4        Float?
  
  comments            String?
  poeAttachments      PortfolioOfEvidence[]
}

model CompetencyScore {
  id                  String               @id @default(uuid())
  agreementId         String
  agreement           PerformanceAgreement @relation(fields: [agreementId], references: [id])
  category            String               // "CORE_PROFESSIONAL" or "PUBLIC_SERVICE_ORIENTATION"
  competencyName      String
  description         String
  weight              Float
  supervisorScore     Float?
  finalScore          Float?
}

model PortfolioOfEvidence {
  id              String               @id @default(uuid())
  agreementId     String
  agreement       PerformanceAgreement @relation(fields: [agreementId], references: [id])
  kpiId           String?
  kpi             KPI?                 @relation(fields: [kpiId], references: [id])
  fileName        String
  fileUrl         String
  uploadedBy      String
  quarter         ReviewQuarter
  isVerified      Boolean              @default(false)
  createdAt       DateTime             @default(now())
}

model ModerationLog {
  id              String               @id @default(uuid())
  agreementId     String
  agreement       PerformanceAgreement @relation(fields: [agreementId], references: [id])
  committee       String               // "DMC" or "MMC"
  originalScore   Float
  moderatedScore  Float
  justification   String
  moderatedBy     String
  createdAt       DateTime             @default(now())
}

model PersonalDevelopmentPlan {
  id              String               @id @default(uuid())
  agreementId     String               @unique
  agreement       PerformanceAgreement @relation(fields: [agreementId], references: [id])
  skillGaps       String
  actionPlan      String
  timeframeMonths Int                  @default(6)
  status          String               @default("IN_PROGRESS")
}

model AuditLog {
  id          String   @id @default(uuid())
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  action      String
  details     String
  ipAddress   String?
  createdAt   DateTime @default(now())
}
