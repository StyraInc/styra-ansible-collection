=======================
Styra.OPA Release Notes
=======================

.. contents:: Topics


v1.0.0
======

Release Summary
---------------

Initial release includes installation of OPA onto Kubernetes with 3 different configuration options:
* Barebones OPA: where policies and data must be pushed to OPA through its API
* Bundle: where OPA will download policies periodically from an HTTP bundle server
* DAS: where OPA is connected to Styra DAS to download policy, upload decisions, update status

