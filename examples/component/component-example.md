# Component Migration Example

The following diff shows what converting a custom Formik text field to a React Hook Form equivalent might look like:

```diff
import {
  ErrorMessage,
  FormGroup,
  InputPrefix,
  InputSuffix,
  Label,
  TextInput,
} from '@trussworks/react-uswds'
import classnames from 'classnames'
-import { useField } from 'formik'
-import { useFocusFirstError } from 'hooks/useFocusFirstError'
-import { useShowErrors } from 'hooks/useShowErrors'
import React, {
  ChangeEventHandler,
  FocusEventHandler,
  ReactNode,
-  useRef,
  useState,
} from 'react'
+import { useController } from 'react-hook-form'

type TextInputProps = Optional<React.ComponentProps<typeof TextInput>, 'id'>

interface ITextFieldProps extends TextInputProps {
  label: ReactNode
  labelClassName?: string
  errorClassName?: string
  labelHint?: string
  hint?: ReactNode
  inputPrefix?: ReactNode
  inputSuffix?: ReactNode
}

export const TextField = ({
  label,
  labelClassName,
  errorClassName,
  labelHint,
  hint,
  inputPrefix,
  inputSuffix,
  onChange,
  onBlur,
  ...textInputProps
}: ITextFieldProps) => {
-  const [fieldProps, metaProps] = useField({
-    name: textInputProps.name,
-    type: textInputProps.type,
-  })
+  const {
+    field: {
+      onChange: hookFormOnChange,
+      onBlur: hookFormOnBlur,
+      ref,
+      value,
+      ...hookFormRemainingProps
+    },
+    fieldState: { invalid, error },
+  } = useController({ name: textInputProps.name })
  const [focused, setFocused] = useState(false)
-  const showError = useShowErrors(textInputProps.name)
-  const showErrorOutline = showError && !focused
-  const textFieldRef = useRef<HTMLInputElement>(null)
-
-  useFocusFirstError(metaProps.error, textFieldRef)
+  const showErrorOutline = invalid && !focused

  const handleBlur: FocusEventHandler<HTMLInputElement> = (e) => {
-    fieldProps.onBlur(e)
+    hookFormOnBlur()
    setFocused(false)
    if (onBlur) {
      onBlur(e)
    }
  }

  const handleChange: ChangeEventHandler<HTMLInputElement> = (e) => {
-    fieldProps.onChange(e)
+    hookFormOnChange(e)
    if (onChange) {
      onChange(e)
    }
  }

  const id = textInputProps.id || textInputProps.name

  const textInput = (
    <TextInput
-      {...fieldProps}
      id={id}
-      value={fieldProps.value || ''}
+      value={value || ''}
      validationStatus={showErrorOutline ? 'error' : undefined}
      onFocus={() => setFocused(true)}
      onChange={handleChange}
      onBlur={handleBlur}
-      inputRef={textFieldRef}
+      inputRef={ref}
      {...textInputProps}
-      onInvalid={(e) => e.preventDefault()}
+      {...hookFormRemainingProps}
    />
  )

  return (
-    <FormGroup error={showError}>
+    <FormGroup error={invalid}>
      <Label
        className={labelClassName}
        hint={labelHint}
-        error={showError}
+        error={invalid}
        htmlFor={id}
      >
        {label}
      </Label>
      {hint && (
        <div
          className="usa-hint"
          id={`${id}.hint`}
          data-testid="text-field-hint"
        >
          {hint}
        </div>
      )}
-      {showError && (
-        <ErrorMessage className={errorClassName}>
-          {metaProps.error}
-        </ErrorMessage>
+      {invalid && (
+        <ErrorMessage className={errorClassName}>{error?.message}</ErrorMessage>
      )}
      {inputSuffix || inputPrefix ? (
        <div
          className={classnames('usa-input-group', {
            'usa-input-group--error': showErrorOutline,
            'is-focused': focused,
          })}
          data-testid="text-field-input-group"
        >
          {inputPrefix && <InputPrefix>{inputPrefix}</InputPrefix>}
          {textInput}
          {inputSuffix && <InputSuffix>{inputSuffix}</InputSuffix>}
        </div>
      ) : (
        textInput
      )}
    </FormGroup>
  )
}
```