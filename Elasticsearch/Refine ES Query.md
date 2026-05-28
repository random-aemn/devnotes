An example of refining an ES query is below.  This example assumes that users can belong to up to 6 nested organizations AND that if they don't
belong to an org, they are associated with an "N/A" value.  T

This checks the user's orgs and updates the defaultQueryString based on what orgs that user is a part of.  
**NOTE: the defaultQueryString will still need a closing parenthesis.  See [Backend Rest Call to ES Index](../Backend REST Call To ES Index.md)
for how the strings are concatenated.

        private String generateSearchQueryForUsersOrgs() {

        OrgsDTO orgDto = userService.getLoggedInUserAllOrgs();

        String defaultQueryString;

        // The user is not a super user so limit to the user's organizations.
        if (orgDto.getTertiaryOrgId().equals(LocalConstants.TERTIARY_ORG_ID_NA)){
            defaultQueryString = String.format("primary_org_id:%d AND secondary_org_id:%d",
                    orgDto.getPrimaryOrgId(), orgDto.getSecondaryOrgId());

        }
        else if (orgDto.getQuaternaryOrgId().equals(LocalConstants.QUATERNARY_ORG_ID_NA)){
            defaultQueryString = String.format("primary_org_id:%d AND secondary_org_id:%d AND tertiary_org_id:%d",
                    orgDto.getPrimaryOrgId(), orgDto.getSecondaryOrgId(),
                    orgDto.getTertiaryOrgId());
        }
        else if (orgDto.getQuinaryOrgId().equals(LocalConstants.QUINARY_ORG_ID_NA)){
            defaultQueryString = String.format("primary_org_id:%d AND secondary_org_id:%d AND tertiary_org_id:%d AND quaternary_org_id:%d",
                    orgDto.getPrimaryOrgId(), orgDto.getSecondaryOrgId(),
                    orgDto.getTertiaryOrgId(), orgDto.getQuaternaryOrgId());
        }
        else if (orgDto.getSenaryOrgId().equals(LocalConstants.SENARY_ORG_ID_NA)){
            defaultQueryString = String.format("primary_org_id:%d AND secondary_org_id:%d AND tertiary_org_id:%d AND quaternary_org_id:%d AND quinary_org_id:%d",
                    orgDto.getPrimaryOrgId(), orgDto.getSecondaryOrgId(),
                    orgDto.getTertiaryOrgId(), orgDto.getQuaternaryOrgId(),
                    orgDto.getQuinaryOrgId());
        }
        else {
            defaultQueryString = String.format("primary_org_id:%d AND secondary_org_id:%d AND tertiary_org_id:%d AND quaternary_org_id:%d AND quinary_org_id:%d AND senary_org_id:%d",
                    orgDto.getPrimaryOrgId(), orgDto.getSecondaryOrgId(),
                    orgDto.getTertiaryOrgId(), orgDto.getQuaternaryOrgId(),
                    orgDto.getQuinaryOrgId(), orgDto.getSenaryOrgId());
        }
        return defaultQueryString;
    }