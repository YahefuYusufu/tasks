Updated Implementation Process for LED Steering Indication

  Step 1: Add New Function to QioAPI class (lib/src/qio_api.dart)

  Location: After line 646 (after removeSteeringActivationButton)

  Future<Map<String, dynamic>> mapSteeringIndicationToLEDs() async {
    final cdcIndicationId = 66071; // CDC.Status.IndicationLEDStatus.ID
    final rightGripLED1Id = 65617; // Right Grip LED1ControlNSI.ID  
    final leftGripLED2Id = 65618;  // Left Grip LED2ControlNSI.ID

    // Map CDC indication status to grip LEDs
    _mappingHandler.updateMappingWithInput(cdcIndicationId, rightGripLED1Id,
  MappingMethod.normal.sign);
    _mappingHandler.updateMappingWithInput(cdcIndicationId, leftGripLED2Id,
  MappingMethod.normal.sign);

    return _mappingHandler.mutateMapping();
  }

  Future<Map<String, dynamic>> removeSteeringIndicationFromLEDs() async {
    final cdcIndicationId = 66071;
    final rightGripLED1Id = 65617;
    final leftGripLED2Id = 65618;

    // Remove mappings
    _mappingHandler.removeFromMapping(cdcIndicationId, rightGripLED1Id, dontMutate: true);
    _mappingHandler.removeFromMapping(cdcIndicationId, leftGripLED2Id, dontMutate: true);

    return _mappingHandler.mutateMapping();
  }

  Step 2: Optional - Extend setSteeringActivationButton (lib/src/qio_api.dart)

  Location: Lines 626-637, modify to optionally enable LED mapping:

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
      await mapSteeringIndicationToLEDs();
    }

    return {'success': true, 'ledMapped': enableLEDIndication};
  }

  Step 3: Key Signal IDs Reference

  - CDC Indication Status: 66071 (CDC.Status.IndicationLEDStatus.ID) - SOURCE
  - Right Grip LED1: 65617 (LED1ControlNSI.ID) - TARGET
  - Left Grip LED2: 65618 (LED2ControlNSI.ID) - TARGET

  Step 4: Usage Examples

  // Map CDC steering indication to LEDs
  await qioAPI.mapSteeringIndicationToLEDs();

  // Enable steering activation AND LED indication
  await qioAPI.setSteeringActivationButton(ControlPos.right1, enableLEDIndication: true);

  // Remove LED indication mapping
  await qioAPI.removeSteeringIndicationFromLEDs();

  Result: When track/wheel-steering becomes active, CDC automatically sets IndicationLEDStatus = 
  1, which lights both grip LEDs. When inactive, LEDs turn off automatically.
