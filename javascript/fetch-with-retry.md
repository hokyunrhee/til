# fetch with retry

```ts
import promiseRetry from "promise-retry";
import { OperationOptions } from "retry";

class FetchError extends Error {
  constructor(message: string) {
    super(message);
    this.name = "FetchError";

    Object.setPrototypeOf(this, FetchError.prototype);
  }
}

type Input = Parameters<typeof fetch>[0];
type Init = NonNullable<Parameters<typeof fetch>[1]> & {
  retryOptions?: OperationOptions;
};

const fetchWithRetry = async (input: Input, init?: Init) => {
  const defaultRetryOptions: OperationOptions = {
    retries: 2,
    factor: 2,
  };

  const { retryOptions = defaultRetryOptions, ...requestInit } = init || {};

  try {
    const result = await promiseRetry(async (retry, attempt) => {
      try {
        return await fetch(input, requestInit).then((response) => {
          if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
          }
          return response;
        });
      } catch (error) {
        if (error instanceof Error) {
          console.error(`Attempt ${attempt} failed:`, error.message);
          retry(error);
        }
        throw error;
      }
    }, retryOptions);

    return result;
  } catch (error) {
    throw new FetchError("All attempt failed");
  }
};
```
