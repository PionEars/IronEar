# Contributing to IronEar

Thank you for your interest. Here is how contributions work.

## Before your first pull request

Contributors sign the IronEar Contributor License Agreement (`CLA.md`) once.
The CLA Assistant check on your first pull request handles it. The
agreement lets IronEar Technologies LLC distribute your contribution under
any license, including the linking exception for the closed inference
engine described in `licenses/LICENSE-firmware.md`. You keep the copyright
to your work.

## What is open and what is not

- Firmware in `firmware/`: GPLv3 with a linking exception. Contributions
  welcome.
- Model in `model/`: Apache 2.0. Contributions to the class configuration
  and tooling welcome; the training data is not published.
- Documentation in `docs/`: CC BY 4.0. Corrections and additions welcome.
- Hardware in `hardware/`: CERN-OHL-S 2.0. Schematics and bill of
  materials at launch; board layout, Gerbers, and the enclosure STEP model
  when units ship. Review comments and fixes welcome.
- Inference engine in `library/`: closed, prebuilt. Bug reports welcome,
  code contributions are not possible.

## How to contribute

1. Open an issue first for anything larger than a small fix, so the change
   can be discussed before you spend time on it.
2. Fork the repository and make your change on a branch.
3. Build the firmware with ESP-IDF 6.0 (`idf.py build`) and test it on a
   board if you can. Say in the pull request what you tested.
4. Sign off each commit (`git commit -s`). The sign-off certifies the
   Developer Certificate of Origin (https://developercertificate.org).
5. Open the pull request. Keep it to one change.

## Style

Follow the layout and naming of the surrounding code. Comments describe
what the code does and why, without author names or dates. Do not leave
commented-out code behind.

## Reporting a security issue

Do not open a public issue. Email steffen@ironear.io with the details.
