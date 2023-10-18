# DigitBoxes

## Approach 1

- `useDigitBoxes`라는 하나의 hook으로 `{ digits, DigitBoxes }`를 반환하여 사용.
- `nextBox.focus()`, `prevbox.focus()` 등이 적용은 되지만 재렌더링되면서 focus가 풀리는 문제가 있음.

```tsx
import { ChangeEvent, useRef, useState } from "react";
import styled from "styled-components";

type Props = {
  numBoxes: number;
};

export default function useDigitBoxes({ numBoxes }: Props) {
  const digitBoxesRef = useRef<HTMLDivElement>(null);

  const [digits, setDigits] = useState<string[]>(new Array(numBoxes).fill(""));

  const onDigitChange = (index: number, value: string) => {
    setDigits((prev) => {
      const newDigits = [...prev];
      newDigits[index] = value;
      // If the user deleted the number in the current digit box, focus the previous box.
      if (value === "") {
        const prevBox = digitBoxesRef.current?.children[
          index - 1
        ] as HTMLInputElement;

        // If the current digit box is not the first box.
        if (index !== 0) {
          prevBox.focus();
        }
      } else {
        // If the user entered a number, focus the next box.
        const nextBox = digitBoxesRef.current?.children[
          index + 1
        ] as HTMLInputElement;

        // If the current digit box is not the last box.
        if (index < numBoxes - 1) {
          nextBox.focus();
        }
      }
      return newDigits;
    });
  };

  const DigitBoxes = () => {
    return (
      <StyledDigitBoxes ref={digitBoxesRef}>
        {digits.map((digit, index) => (
          <DigitBox
            key={index}
            index={index}
            value={digit}
            onChange={onDigitChange}
          />
        ))}
      </StyledDigitBoxes>
    );
  };

  const DigitBox = ({
    index,
    value,
    onChange,
  }: {
    index: number;
    value: string;
    onChange: (index: number, value: string) => void;
  }) => {
    return (
      <StyledDigitBox
        type="tel"
        pattern="[0-9]{1}"
        value={value}
        onChange={(e: ChangeEvent<HTMLInputElement>) =>
          onChange(index, e.target.value)
        }
        required
      />
    );
  };

  return { digits, DigitBoxes };
}
```
