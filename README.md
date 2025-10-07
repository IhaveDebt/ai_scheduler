/**
 * AI Workflow Scheduler (ai_scheduler.ts)
 *
 * A dependency-aware task orchestrator written in TypeScript — simulates an ML pipeline workflow.
 *
 * Run:
 *   ts-node src/ai_scheduler.ts
 */

type Job = {
  id: string;
  name: string;
  dependsOn: string[];
  execute: () => Promise<void>;
};

class Scheduler {
  private jobs: Map<string, Job> = new Map();
  private completed = new Set<string>();

  addJob(job: Job) {
    if (this.jobs.has(job.id)) throw new Error(`Duplicate job ID: ${job.id}`);
    this.jobs.set(job.id, job);
  }

  async run() {
    console.log("=== AI Workflow Scheduler ===");
    const pending = new Set(this.jobs.keys());
    while (pending.size) {
      let progress = false;
      for (const id of Array.from(pending)) {
        const job = this.jobs.get(id)!;
        if (job.dependsOn.every(dep => this.completed.has(dep))) {
          console.log(`Running: ${job.name}`);
          try {
            await job.execute();
            this.completed.add(id);
            pending.delete(id);
            progress = true;
          } catch (err) {
            console.error(`Job ${job.name} failed:`, err);
            // For a scheduler demo, we stop on failure. Production: add retries/backoff.
            throw err;
          }
        }
      }
      if (!progress) throw new Error("Circular dependency detected or missing dependency!");
    }
    console.log("✅ All jobs completed!");
  }
}

// Example use
async function main() {
  const scheduler = new Scheduler();

  scheduler.addJob({
    id: "data",
    name: "Fetch Training Data",
    dependsOn: [],
    execute: async () => {
      await new Promise(r => setTimeout(r, 1000));
      console.log("Data fetched.");
    }
  });

  scheduler.addJob({
    id: "preprocess",
    name: "Preprocess Data",
    dependsOn: ["data"],
    execute: async () => {
      await new Promise(r => setTimeout(r, 700));
      console.log("Data preprocessed.");
    }
  });

  scheduler.addJob({
    id: "train",
    name: "Train Model",
    dependsOn: ["preprocess"],
    execute: async () => {
      await new Promise(r => setTimeout(r, 2000));
      console.log("Model trained.");
    }
  });

  scheduler.addJob({
    id: "validate",
    name: "Validate Model",
    dependsOn: ["train"],
    execute: async () => {
      await new Promise(r => setTimeout(r, 500));
      console.log("Model validated.");
    }
  });

  scheduler.addJob({
    id: "deploy",
    name: "Deploy Model",
    dependsOn: ["validate"],
    execute: async () => {
      await new Promise(r => setTimeout(r, 600));
      console.log("Model deployed.");
    }
  });

  await scheduler.run();
}

if (require.main === module) {
  main().catch(err => {
    console.error("Scheduler failed:", err);
    process.exit(1);
  });
}
