# ── Build stage ─────────────────────────────────────────────────────────────
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app

# Download dependencies first so this layer is cached between builds
COPY pom.xml .
RUN mvn -B dependency:go-offline

COPY src ./src
# Tests run in the CI "test" job, so they're skipped here
RUN mvn -B clean package -DskipTests

# ── Runtime stage ───────────────────────────────────────────────────────────
FROM eclipse-temurin:21-jre
WORKDIR /app

# Run as a non-root user
RUN groupadd --system app && useradd --system --gid app app

COPY --from=build /app/target/*.jar app.jar

# Safe defaults so the container boots without any secrets:
#  - AGENT_PROVIDER=scripted -> offline fake model, no LLM key needed
#  - empty Razorpay values   -> offline stub payment gateway
# Override any of these with `docker run -e ...` (DB_URL, DB_USER, DB_PASSWORD,
# AGENT_PROVIDER, GROQ_API_KEY, GEMINI_API_KEY, RAZORPAY_KEY_ID, ...).
ENV PORT=8080 \
    AGENT_PROVIDER=scripted \
    RAZORPAY_KEY_ID="" \
    RAZORPAY_KEY_SECRET="" \
    RAZORPAY_WEBHOOK_SECRET="" \
    RAZORPAY_CALLBACK_URL="" \
    JAVA_OPTS="-XX:MaxRAMPercentage=75.0"

USER app
EXPOSE 8080

ENTRYPOINT ["sh", "-c", "exec java $JAVA_OPTS -jar app.jar"]
