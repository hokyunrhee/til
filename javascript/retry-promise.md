# retry promise

`retry-promise.ts`

```ts
import promiseRetry from "promise-retry";
import { OperationOptions } from "retry";

export class RetryError extends Error {
  constructor(message: string) {
    super(message);
    this.name = "RetryError";

    Object.setPrototypeOf(this, RetryError.prototype);
  }
}

export const retryPromise = async <T>(
  asyncFunction: () => Promise<T>,
  retryOptions?: OperationOptions
): Promise<T> => {
  const defaultRetryOptions = {
    retries: 2,
    factor: 2,
  } satisfies OperationOptions;

  const retryOpts = { ...defaultRetryOptions, ...retryOptions };

  try {
    return await promiseRetry(async (retry) => {
      try {
        return await asyncFunction();
      } catch (error) {
        return retry(error);
      }
    }, retryOpts);
  } catch (error) {
    throw new RetryError("All retries failed");
  }
};
```

`retry-promise.test.ts`

```ts
import { describe, it, expect, vi } from "vitest";
import { retryPromise, RetryError } from "./retry-promise";

describe("retryPromise", () => {
  it("should resolve immediately if the async function succeeds on first try", async () => {
    const successFn = vi.fn().mockResolvedValue("success");

    const result = await retryPromise(successFn);

    expect(result).toBe("success");
    expect(successFn).toHaveBeenCalledTimes(1);
  });

  it("should retry and eventually succeed", async () => {
    const successAfterRetry = vi
      .fn()
      .mockRejectedValueOnce(new Error("First failure"))
      .mockRejectedValueOnce(new Error("Second failure"))
      .mockResolvedValue("success");

    const result = await retryPromise(successAfterRetry);

    expect(result).toBe("success");
    expect(successAfterRetry).toHaveBeenCalledTimes(3);
  });

  it("should throw RetryError after all retries fail", async () => {
    const error = new Error("Persistent error");
    const alwaysFailFn = vi.fn().mockRejectedValue(error);

    await expect(() => retryPromise(alwaysFailFn)).rejects.toThrowError(
      RetryError
    );
    expect(alwaysFailFn).toHaveBeenCalledTimes(3);
  });

  it("should handle concurrent retries independently", async () => {
    const error = new Error("Test error");
    const failingFn1 = vi.fn().mockRejectedValue(error);
    const failingFn2 = vi.fn().mockRejectedValue(error);

    const promise1 = retryPromise(failingFn1);
    const promise2 = retryPromise(failingFn2);

    const [result1, result2] = await Promise.allSettled([promise1, promise2]);

    expect(result1.status).toEqual("rejected");
    expect(result2.status).toEqual("rejected");
    expect(failingFn1).toHaveBeenCalledTimes(3);
    expect(failingFn2).toHaveBeenCalledTimes(3);
  });
});
```
