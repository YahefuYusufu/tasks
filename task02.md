 Complete Implementation Process for LED Button Mapping

  Step 1: Add New Function to QioAPI class (lib/src/qio_api.dart)

  Location: After line 646 (after removeSteeringActivationButton)

  Future<Map<String, dynamic>> setSteeringLEDIndication(bool enable) async {
    final cdcLEDId = 66071; // CDC.Status.IndicationLEDStatus.ID
    final rightGripLED1Id = 65617; // Right Grip LED1ControlNSI.ID  
    final rightGripLED2Id = 65618; // Right Grip LED2ControlNSI.ID

  if (enable) {
      // Map CDC LED status to both grip LEDs
      _mappingHandler.updateMappingWithInput(cdcLEDId, rightGripLED1Id,
  MappingMethod.normal.sign);
      _mappingHandler.updateMappingWithInput(cdcLEDId, rightGripLED2Id,
  MappingMethod.normal.sign);
    } else {
      // Remove mappings
      _mappingHandler.removeFromMapping(cdcLEDId, rightGripLED1Id, dontMutate: true);
      _mappingHandler.removeFromMapping(cdcLEDId, rightGripLED2Id, dontMutate: true);
    }

   return _mappingHandler.mutateMapping();
  }

  Step 2: Extend existing setSteeringActivationButton function (lib/src/qio_api.dart)

  Location: Lines 626-637, replace with:

  Future<Map<String, dynamic>> setSteeringActivationButton(
      ControlPos pos, {bool enableLEDIndication = false}) async {
    final button = _getControlAtPos(pos);
    if (button is! Button) {
      throw 'Only buttons can be used for steering activation. Got: ${button.runtimeType}';
    }
    final int btnId = button.getStateId(null);
    _mappingHandler.updateMappingWithInput(
        btnId, _mappingHandler.activationNsiId, MappingMethod.normal.sign);

  await _mappingHandler.mutateMapping();

   // Optional LED indication mapping
    if (enableLEDIndication) {
      await setSteeringLEDIndication(true);
    }

   return {'success': true, 'ledMapped': enableLEDIndication};
  }

  Step 3: Add LED removal to removeSteeringActivationButton (lib/src/qio_api.dart)

  Location: Lines 639-646, replace with:

  Future<Map<String, dynamic>> removeSteeringActivationButton(
      ControlPos pos, {bool removeLEDIndication = true}) async {
    final button = _getControlAtPos(pos);
    final int btnId = button.getStateId(null);

   final result = await _mappingHandler.removeFromMapping(
        btnId, _mappingHandler.activationNsiId);

   // Optional LED indication removal
    if (removeLEDIndication) {
      await setSteeringLEDIndication(false);
    }

   return result;
  }

  Step 4: Add LED status getter function (lib/src/qio_api.dart)

  Location: After the new setSteeringLEDIndication function

  Future<Map<String, dynamic>> getSteeringLEDStatus() async {
    final cdcLEDId = 66071;
    final rightGripLED1Id = 65617;
    final rightGripLED2Id = 65618;

  final resp = await _comms.query([cdcLEDId, rightGripLED1Id, rightGripLED2Id]);

   return {
      'cdcIndicationLED': findValueById(resp, cdcLEDId)['Data'],
      'rightGripLED1': findValueById(resp, rightGripLED1Id)['Data'],
      'rightGripLED2': findValueById(resp, rightGripLED2Id)['Data'],
    };
  }

  Step 5: Key Signal IDs Reference

  Constants to use:
  - CDC Indication LED: 66071 (CDC.Status.IndicationLEDStatus.ID)
  - Right Grip LED1: 65617 (Right Grip LED1ControlNSI.ID)
  - Right Grip LED2: 65618 (Right Grip LED2ControlNSI.ID)

  Step 6: Usage Examples

  // Enable steering activation with LED indication
  await qioAPI.setSteeringActivationButton(ControlPos.right1, enableLEDIndication: true);

  // Just enable LED indication separately
  await qioAPI.setSteeringLEDIndication(true);

  // Check LED status
  final status = await qioAPI.getSteeringLEDStatus();

  // Remove with LED cleanup
  await qioAPI.removeSteeringActivationButton(ControlPos.right1, removeLEDIndication: true);

  This creates the mapping: CDC.Status.IndicationLEDStatus → Right Grip LED1 & LED2 for
  track/wheel-steering indication.
