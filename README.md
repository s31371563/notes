const plans = [
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
  }
];

const planMappings = [
  {
    "basePlanId":null,
    "mappedPlanId":"p1"
  },
  {
    "basePlanId":null,
    "mappedPlanId":"p2"
  },
  {
    "basePlanId":"p2",
    "mappedPlanId":"p3"
  },
  {
    "basePlanId":null,
    "mappedPlanId":"p5"
  },
  {
    "basePlanId":"p5",
    "mappedPlanId":"p6"
  }
];


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
import java.util.List;

public class PlanMapper {
    public static void main(String[] args) {
        List<PlanDTO> plans = new ArrayList<>();
        plans.add(new PlanDTO("p1", "plan Name", "Q1"));
        plans.add(new PlanDTO("p2", "plan Name 2", "Q2"));
        plans.add(new PlanDTO("p3", "plan Name 3", "Q4"));

        List<PlanMappingDTO> planMappings = new ArrayList<>();
        planMappings.add(new PlanMappingDTO(null, "p1"));
        planMappings.add(new PlanMappingDTO(null, "p2"));
        planMappings.add(new PlanMappingDTO("p2", "p3"));
        planMappings.add(new PlanMappingDTO("p5", "p6"));

        List<BasePlanDTO> basePlans = new ArrayList<>();

        // Filter plans to get base plans
        for (PlanDTO plan : plans) {
            boolean isBasePlan = true;
            for (PlanMappingDTO planMapping : planMappings) {
                if (planMapping.getMappedPlanId().equals(plan.getPlanId()) && planMapping.getBasePlanId() != null) {
                    isBasePlan = false;
                    break;
                }
            }
            if (isBasePlan) {
                List<LegacyPlanMappingDTO> legacyPlanMappings = new ArrayList<>();
                legacyPlanMappings.add(new LegacyPlanMappingDTO(plan.getPlanId(), plan.getPlanName(), plan.getQuarter()));
                BasePlanDTO basePlan = new BasePlanDTO(plan.getPlanName(), legacyPlanMappings);

                // Find regional plans for this base plan
                for (PlanMappingDTO planMapping : planMappings) {
                    if (planMapping.getBasePlanId() != null && planMapping.getBasePlanId().equals(plan.getPlanId())) {
                        PlanDTO regionalPlan = plans.stream()
                                .filter(p -> p.getPlanId().equals(planMapping.getMappedPlanId()))
                                .findFirst()
                                .orElse(null);
                        if (regionalPlan != null) {
                            legacyPlanMappings.add(new LegacyPlanMappingDTO(regionalPlan.getPlanId(), regionalPlan.getPlanName(), regionalPlan.getQuarter()));
                        }
                    }
                }

                basePlans.add(basePlan);
            }
        }

        // Print base plans and their legacy plan mappings
        for (BasePlanDTO basePlan : basePlans) {
            System.out.println("Base Plan Name: " + basePlan.getBasePlanName());
            System.out.println("Legacy Plan Mappings:");
            for (LegacyPlanMappingDTO legacyPlanMapping : basePlan.getLegacyPlanMapping()) {
                System.out.println("Plan ID: " + legacyPlanMapping.getPlanId());
                System.out.println("Plan Name: " + legacyPlanMapping.getPlanName());
                System.out.println("Quarter: " + legacyPlanMapping.getQuarter());
            }
            System.out.println();
        }
    }
}

