# Full Example — Confirm Page Stepper

## `app/confirm/components.tsx`

```tsx
'use client';

import { Suspense, use } from 'react';

export type Step = {
  title: string;
  description: string;
  work: Promise<any>;
};

type StepProps = Step & {
  isLast: boolean;
};

export function StepComponent({ title, description, work, isLast }: StepProps) {
  return (
    <li className={`ml-6 ${isLast ? '' : 'mb-10'} relative`}>
      <span className="absolute -left-4 flex h-8 w-8 items-center justify-center rounded-full bg-white ring-4 ring-white dark:bg-gray-700 dark:ring-gray-900">
        <Suspense fallback={<StepIcon status="in-progress" />}>
          <Asyncable work={work}>
            <StepIcon status="done" />
          </Asyncable>
        </Suspense>
      </span>
      <Suspense fallback={<Title disabled>{title}</Title>}>
        <Asyncable work={work}>
          <Title>{title}</Title>
        </Asyncable>
      </Suspense>
      <p className="text-sm text-gray-600 dark:text-gray-300">{description}</p>
    </li>
  );
}

const Asyncable = ({ work, children }: { work: Promise<any>; children: React.ReactNode }) => {
  use(work);
  return <>{children}</>;
};

const Title = ({ children, disabled }: { children: React.ReactNode; disabled?: boolean }) => {
  return (
    <h3 className={`${disabled ? 'text-gray-400' : 'text-green-600 dark:text-green-400'} font-medium leading-tight`}>
      {children}
    </h3>
  );
};

const StepIcon = ({ status }: { status: 'in-progress' | 'done' }) => {
  return status === 'done' ? '✅' : '⏳';
};
```

## `app/confirm/sequential.ts`

```ts
export function unsafe_createSequentialProcesses<T extends any[], R>(
  ...processes: [(arg?: any) => Promise<T[0]>, ...((arg: any) => Promise<any>)[]]
): Promise<R>[] {
  return processes.reduce((acc, process, index) => {
    if (index === 0) return [process(undefined)];
    return [...acc, acc[acc.length - 1].then(process)];
  }, [] as Promise<any>[]);
}
```

## `app/confirm/page.tsx`

```tsx
import { StepComponent } from './components';
import { unsafe_createSequentialProcesses } from './sequential';

const delay = (ms: number) => new Promise((res) => setTimeout(res, ms));

async function firstProcess(id: string) {
  await delay(2000);
  return `Verified ID: ${id}`;
}

async function secondProcess(input: any) {
  await delay(2000);
  return `Generated PDF from: ${input}`;
}

async function thirdProcess(input: any) {
  await delay(2000);
  return `Uploaded to S3: ${input}`;
}

export default async function ConfirmPage({ params: { id } }: { params: { id: string } }) {
  const [first, second, third] = unsafe_createSequentialProcesses(
    () => firstProcess(id),
    secondProcess,
    thirdProcess
  );

  return (
    <div className="flex flex-col space-y-4 px-4 py-8">
      <ol className="relative border-l border-gray-200 dark:border-gray-700">
        <StepComponent
          title="Verify Payment"
          description="Wait for server confirmation of payment."
          work={first}
          isLast={false}
        />
        <StepComponent
          title="Generate PDF"
          description="Creating personalized PDF."
          work={second}
          isLast={false}
        />
        <StepComponent
          title="Upload to S3"
          description="Uploading file to cloud storage."
          work={third}
          isLast={true}
        />
      </ol>
    </div>
  );
}
```
