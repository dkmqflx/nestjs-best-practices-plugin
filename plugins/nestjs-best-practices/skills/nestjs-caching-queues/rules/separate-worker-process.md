---
title: Run Queue Consumers as a Separate Process for Scaling and Isolation
impact: MEDIUM
tags: queues, bullmq, workers, scaling, sandboxed-processors, deployment
---

## Run Queue Consumers as a Separate Process for Scaling and Isolation

Running producers (the API) and consumers (the workers) in the same process is convenient but limiting. CPU-heavy jobs block the same event loop that serves HTTP requests, a crash in a job can take down the API, and you cannot scale the two independently — the workload that needs 10 worker replicas and the API that needs 2 are forced to scale together.

Separate them. The most robust option is a **dedicated worker deployment**: the same Nest codebase started with a worker-only module (registering the queues and `@Processor` consumers, but not HTTP controllers), so producers and consumers scale as independent services sharing one Redis. For CPU-bound jobs specifically, BullMQ also supports **sandboxed processors** that run each job in a forked child process — note these run outside Nest's DI container, so the processor file must construct its own dependencies.

**Incorrect:**

```typescript
// API and CPU-bound consumers in one process: heavy jobs stall HTTP,
// and you can't scale workers without scaling the API too.
@Module({
  imports: [BullModule.registerQueue({ name: 'video' })],
  controllers: [VideoController], // serves requests
  providers: [VideoService, VideoConsumer], // and runs the heavy processor
})
export class AppModule {}
```

**Correct:**

```typescript
// worker.module.ts — queues + consumers only, no HTTP controllers.
@Module({
  imports: [
    BullModule.forRoot({ connection: { host: 'redis', port: 6379 } }),
    BullModule.registerQueue({ name: 'video' }),
  ],
  providers: [VideoConsumer],
})
export class WorkerModule {}

// worker.ts — boot a context (no HTTP listener); deploy/scale this separately.
async function bootstrap() {
  await NestFactory.createApplicationContext(WorkerModule);
}
bootstrap();
```

```typescript
// Alternative for CPU-bound work: sandboxed (forked) processor — runs outside DI.
BullModule.registerQueue({
  name: 'video',
  processors: [join(__dirname, 'video.processor.js')],
});
```
