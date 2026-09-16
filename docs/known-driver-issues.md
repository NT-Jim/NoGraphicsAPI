# Known driver issues

These are locally reproduced observations, not vendor-confirmed root causes.

## NVIDIA 596.99: stale address-based texture readback

Observed on an RTX 4090 with NVIDIA 596.99, with both sequential and parallel command recording.
An upload through `vkCmdCopyMemoryToImageKHR` followed by `vkCmdCopyImageToMemoryKHR` returned stale
data despite a transfer-write to transfer-read barrier. The issue also reproduced without validation.

A timeline wait between separate upload and readback submissions works. A full Vulkan memory
dependency also worked in isolation; native `vkCmdCopyImageToBuffer2` readback worked in the comparison.
The parallel texture test uses an explicit timeline wait. No driver-specific barrier widening is applied
by the graphics API.

## NVIDIA 596.99: copy-queue timestamp resolution loses the device

On an RTX 4090, resolving timestamps with `vkCmdCopyQueryPoolResultsToMemoryKHR` on a copy-only queue
returns `VK_ERROR_DEVICE_LOST`, with or without validation. Timestamp writes without the resolve complete;
buffer/texture transfers and general/compute-queue timestamp resolution also pass.
The [Vulkan command contract](https://docs.vulkan.org/refpages/latest/refpages/source/vkCmdCopyQueryPoolResultsToMemoryKHR.html)
permits transfer-only queues.

Avoid `write_timestamp` on copy-only queues with this driver. No workaround or silent timestamp suppression
is applied by the library. The regular queue-family tests omit copy-queue markers; reproduce the failure
explicitly with `build-msvc/tests/Debug/test_queue_families.exe --copy-timestamps`.
