# Operations/Production - Data Loaders

### NIHMS API Token Generation

eRA Commons Login: https://public.era.nih.gov/commonsplus/public/login.era

API Token Generation*: PMC
*select using eRA commons to login

System Owner: The system owner is the NIH, but account management is delegated to a University’s Office of Sponsored
Research. In JHU’s case, this is: Johns Hopkins University Research Administration (JHURA). For any other university
setting up their own Nihms-data loader, it will be the Office of Sponsored Research that creates the account for the
API key.

Account Setup: In the case of JHU, the account needs to be set up by JHU Research Administration (JHURA). They will need
to have the following permissions in order to create an account: SO, AO, AA, or BO and they cannot be affiliated with
more than one institution.

If any modifications to the account need to be made, such as changing the associated email with the account, it will
need to be done via the eRA administrator.

Association with Pass: This account is used by the Nihms-loader in Pass-Support. It is responsible for generating an
API token, with a lifespan of 3 months, used to pull PACM data.

Other Documentation:
https://www.ncbi.nlm.nih.gov/pmc/utils/pacm/static/pacm-user-guide.pdf
