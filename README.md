# CSharpLoops
Learn Loops


class ActionYourActionName(Action):
    def name(self) -> Text:
        return "action_your_action_name"

    async def run(self, dispatcher: CollectingDispatcher, tracker: Tracker, domain: Dict[Text, Any]) -> List[EventType]:
        # Extract user message payload
        latest_message = tracker.latest_message
        payload = latest_message.get("text") or latest_message.get("payload") or {}
        
        logger.info(f"Latest message payload: {payload}")

        # Check if the form is already submitted
        selected_desk = None

        # Try to get selected value from payload
        if isinstance(payload, dict):
            selected_desk = payload.get("DESK")
        elif isinstance(payload, str):
            # If payload comes as a JSON string
            try:
                import json
                payload_dict = json.loads(payload)
                selected_desk = payload_dict.get("DESK")
            except json.JSONDecodeError:
                pass

        logger.info(f"Selected Desk: {selected_desk}")

        if selected_desk:
            # ✅ User has submitted the form
            dispatcher.utter_message(text=f"You selected: {selected_desk}")
            return []

        # 🔄 First time: Show the form
        # Get components (simulate your data source)
        components = await self.get_components()

        components_data = [{'title': name, 'value': id} for name, id in components.items()]
        payload = {
            "DESK": None,
            "desks": components_data,
            "intent": "pi_plan_intake"
        }

        # Send adaptive card
        dispatcher.utter_message(
            text="Please select a desk from the dropdown:",
            attachment=self.PI_PLAN_INTAKE_FORM.populate_attachment(data=payload)
        )

        return []

    # Simulate your components fetching method
    async def get_components(self):
        return {
            "Desk A": "desk_a",
            "Desk B": "desk_b",
            "Desk C": "desk_c"
        }
