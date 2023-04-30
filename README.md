
[
	{
		"planId":"p1",
		"planName":"plan Name",
		"quarter": "Q1"
	},
	{
		"planId":"p2",
		"planName":"plan Name 2",
		"quarter": "Q2"
	},
	{
		"planId":"p3",
		"planName":"plan Name 3",
		"quarter": "Q4"
	},
]

[
	{
		"basePlanId":null, //basePlanId may be null or empty
		"mappedPlanId":"p1"
	},
	{
		"basePlanId":"p2",
		"mappedPlanId":"p3"
	}
]


I need to use PlanMapping & PlanCodes to generate json as below.
[
	{
		"basePlanName":"plan Name",
		"legacyPlanMapping": [
			{
				"planId": "p1",
				"planName": "plan Name",
				"quater":"Q1"
			}
		]
	},
	{
		"basePlanName":"plan Name 2",
		"legacyPlanMapping": [
			{
				"planId": "p2",
				"planName": "plan Name 2",
				"quater":"Q2"
			},
			{
				"planId": "p3",
				"planName": "plan Name 3",
				"quater":"Q4"
			},
		]
	}
]

result:

[
    {
        "basePlanName": "plan Name",
        "legacyPlanMapping": [
            {
                "planId": "p1",
                "planName": "plan Name",
                "quarter": "Q1"
            }
        ]
    },
    {
        "basePlanName": "plan Name 2",
        "legacyPlanMapping": [
            {
                "planId": "p3",
                "planName": "plan Name 3",
                "quarter": "Q4"
            }
        ]
    }
]



import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class Example {

	public static void main(String[] args) {
		List<PlanMapping> planMappings = mockPlanMappings();
		List<PlanCodes> planCodes = mockPlanCodes();

		Map<String, PlanCodes> planMap = new HashMap<>();
		for (PlanCodes plan : planCodes) {
			planMap.put(plan.getPlanId(), plan);
		}

		List<BasePlan> basePlans = new ArrayList<>();
		for (PlanMapping mapping : planMappings) {
			if (mapping.getBasePlanId() == null || mapping.getBasePlanId().isEmpty()) {
				PlanCodes plan = planMap.get(mapping.getMappedPlanId());
				basePlans.add(new BasePlan(plan.getPlanName(), List.of(plan)));
			} else {
				PlanCodes basePlan = planMap.get(mapping.getBasePlanId());
				List<PlanCodes> mappedPlans = new ArrayList<>();
				for (PlanMapping innerMapping : planMappings) {
					if (innerMapping.getBasePlanId() != null
							&& innerMapping.getBasePlanId().equals(basePlan.getPlanId())) {
						PlanCodes mappedPlan = planMap.get(innerMapping.getMappedPlanId());
						mappedPlans.add(mappedPlan);
					}
				}
				basePlans.add(new BasePlan(basePlan.getPlanName(), mappedPlans));
			}
		}

		String json = toJSON(basePlans);
		System.out.println(json);
	}

	private static List<PlanMapping> mockPlanMappings() {
		List<PlanMapping> planMappings = new ArrayList<>();
		planMappings.add(new PlanMapping(null, "p1"));
		planMappings.add(new PlanMapping("p2", "p3"));
		return planMappings;
	}

	private static List<PlanCodes> mockPlanCodes() {
		List<PlanCodes> planCodes = new ArrayList<>();
		planCodes.add(new PlanCodes("p1", "plan Name", "Q1"));
		planCodes.add(new PlanCodes("p2", "plan Name 2", "Q2"));
		planCodes.add(new PlanCodes("p3", "plan Name 3", "Q4"));
		return planCodes;
	}

	private static String toJSON(List<BasePlan> basePlans) {
		StringBuilder sb = new StringBuilder("[");
		for (int i = 0; i < basePlans.size(); i++) {
			BasePlan basePlan = basePlans.get(i);
			sb.append("{");
			sb.append("\"basePlanName\":\"").append(basePlan.getBasePlanName()).append("\",");
			sb.append("\"legacyPlanMapping\":[");
			for (int j = 0; j < basePlan.getLegacyPlanMapping().size(); j++) {
				PlanCodes plan = basePlan.getLegacyPlanMapping().get(j);
				sb.append("{");
				sb.append("\"planId\":\"").append(plan.getPlanId()).append("\",");
				sb.append("\"planName\":\"").append(plan.getPlanName()).append("\",");
				sb.append("\"quarter\":\"").append(plan.getQuarter()).append("\"");
				sb.append("}");
				if (j < basePlan.getLegacyPlanMapping().size() - 1) {
					sb.append(",");
				}
			}
			sb.append("]}");
			if (i < basePlans.size() - 1) {
				sb.append(",");
			}
		}
		sb.append("]");
		return sb.toString();
	}

}

class BasePlan {
	private String basePlanName;
	private List<PlanCodes> legacyPlanMapping;

	public BasePlan(String basePlanName, List<PlanCodes> legacyPlanMapping) {
		this.basePlanName = basePlanName;
		this.legacyPlanMapping = legacyPlanMapping;
	}

	public String getBasePlanName() {
		return basePlanName;
	}

	public void setBasePlanName(String basePlanName) {
		this.basePlanName = basePlanName;
	}

	public List<PlanCodes> getLegacyPlanMapping() {
		return legacyPlanMapping;
	}

	public void setLegacyPlanMapping(List<PlanCodes> legacyPlanMapping) {
		this.legacyPlanMapping = legacyPlanMapping;
	}
}

class PlanCodes {
	private String planId;
	private String planName;
	private String quarter;

	public PlanCodes(String planId, String planName, String quarter) {
		this.planId = planId;
		this.planName = planName;
		this.quarter = quarter;
	}

	public String getPlanId() {
		return planId;
	}

	public void setPlanId(String planId) {
		this.planId = planId;
	}

	public String getPlanName() {
		return planName;
	}

	public void setPlanName(String planName) {
		this.planName = planName;
	}

	public String getQuarter() {
		return quarter;
	}

	public void setQuarter(String quarter) {
		this.quarter = quarter;
	}
}

class PlanMapping {
	private String basePlanId;
	private String mappedPlanId;

	public PlanMapping(String basePlanId, String mappedPlanId) {
		this.basePlanId = basePlanId;
		this.mappedPlanId = mappedPlanId;
	}

	public String getBasePlanId() {
		return basePlanId;
	}

	public void setBasePlanId(String basePlanId) {
		this.basePlanId = basePlanId;
	}

	public String getMappedPlanId() {
		return mappedPlanId;
	}

	public void setMappedPlanId(String mappedPlanId) {
		this.mappedPlanId = mappedPlanId;
	}
}
