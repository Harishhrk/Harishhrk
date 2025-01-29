
import { render } from '../../../util/testUtils'
import { fireEvent } from '@testing-library/react'
import { language } from '../../../helper/languages'
import AddReviewComponent from '../AddReviewComponent'
import { Grid } from '@connect/grid'
import ViewReviewComponent from '../ViewReviewComponent'
import * as reviewsApiHook from '../../../services/reviewsApi'
import { reviewsHookMockData } from '../../../__mocks__/reviewsData'
import * as ReviewsApiHook from '../../../services/reviewsApi'
import { AddReviewPayload } from '../types/types'

const {
  REVIEW_SCREEN_YES_OPTION,
  REVIEW_SCREEN_NO_OPTION,
  REVIEW_SCREEN_COMMENTS_LABEL,
  REQUESTED_CHANGES_TITLE,
  NO_COMMENTS_MSG,
  MIN_LENGTH_COMMENTS_ERROR_MESSAGE,
  MAX_LENGTH_COMMENTS_ERROR_MESSAGE,
  EMPTY_COMMENTS_ERROR_MESSAGE,
  INTERNAL_SERVER_ERROR,
} = language

jest.mock('react-router-dom', () => ({
  ...jest.requireActual('react-router-dom'),
  useParams: () => ({
    stepId: 's1',
    flowId: '221',
  }),
}))

describe('AddReviewScreen Component', () => {
  describe('Initial Rendering', () => {
    it('renders correctly with default "Yes" option selected', () => {
      const mockSetReviewPayload = jest.fn()
      const { getByLabelText } = render(
        <Grid>
          <AddReviewComponent
            onValidationChange={jest.fn()}
            setReviewPayload={mockSetReviewPayload}
          />
        </Grid>
      )

      const yesRadio = getByLabelText(REVIEW_SCREEN_YES_OPTION)
      const noRadio = getByLabelText(REVIEW_SCREEN_NO_OPTION)

      expect(yesRadio).toBeChecked()
      expect(noRadio).not.toBeChecked()
    })
  })

  describe('Radio Button Interactions', () => {
    it('shows textarea when "No" option is selected', () => {
      const mockSetReviewPayload = jest.fn()
      const { getByLabelText } = render(
        <Grid>
          <AddReviewComponent
            onValidationChange={jest.fn()}
            setReviewPayload={mockSetReviewPayload}
          />
        </Grid>
      )

      const noRadio = getByLabelText(REVIEW_SCREEN_NO_OPTION)
      fireEvent.click(noRadio)

      const textarea = getByLabelText(REVIEW_SCREEN_COMMENTS_LABEL)
      expect(textarea).toBeInTheDocument()
    })
  })

  describe('Textarea Functionality', () => {
    it('allows text input when "No" is selected', () => {
      const mockSetReviewPayload = jest.fn()
      const { getByLabelText } = render(
        <Grid>
          <AddReviewComponent
            onValidationChange={jest.fn()}
            setReviewPayload={mockSetReviewPayload}
          />
        </Grid>
      )

      const noRadio = getByLabelText(REVIEW_SCREEN_NO_OPTION)
      fireEvent.click(noRadio)

      const testComment = 'Test comment'

      const textarea = getByLabelText(REVIEW_SCREEN_COMMENTS_LABEL)
      fireEvent.change(textarea, { target: { value: testComment } })

      expect(textarea).toHaveValue(testComment)
    })
  })

  describe('Textarea error functionality', () => {
    it('text area shows error when empty comments validation fails', () => {
      const mockSetReviewPayload = jest.fn()
      const { getByLabelText } = render(
        <Grid>
          <AddReviewComponent
            onValidationChange={jest.fn()}
            setReviewPayload={mockSetReviewPayload}
          />
        </Grid>
      )

      const noRadio = getByLabelText(REVIEW_SCREEN_NO_OPTION)
      fireEvent.click(noRadio)

      const testComment = ''

      const textarea = getByLabelText(REVIEW_SCREEN_COMMENTS_LABEL)
      fireEvent.change(textarea, { target: { value: testComment } })
      fireEvent.blur(textarea)

      expect(textarea).toHaveValue(testComment)
    })

    it('text area shows error when min length validation fails', () => {
      const mockSetReviewPayload = jest.fn()
      const { getByLabelText } = render(
        <Grid>
          <AddReviewComponent
            onValidationChange={jest.fn()}
            setReviewPayload={mockSetReviewPayload}
          />
        </Grid>
      )

      const noRadio = getByLabelText(REVIEW_SCREEN_NO_OPTION)
      fireEvent.click(noRadio)

      const testComment = 'ab'

      const textarea = getByLabelText(REVIEW_SCREEN_COMMENTS_LABEL)
      fireEvent.change(textarea, { target: { value: testComment } })
      fireEvent.blur(textarea)

      expect(textarea).toHaveValue(testComment)
    })

    it('text area shows error when max length validation fails', () => {
      const mockSetReviewPayload = jest.fn()
      const { getByLabelText } = render(
        <Grid>
          <AddReviewComponent
            onValidationChange={jest.fn()}
            setReviewPayload={mockSetReviewPayload}
          />
        </Grid>
      )

      const noRadio = getByLabelText(REVIEW_SCREEN_NO_OPTION)
      fireEvent.click(noRadio)

      const testComment = 'x'.repeat(2501)

      const textarea = getByLabelText(REVIEW_SCREEN_COMMENTS_LABEL)
      fireEvent.change(textarea, { target: { value: testComment } })
      fireEvent.blur(textarea)

      expect(textarea).toHaveValue(testComment)
    })

    it('should validate comments on change after blur', () => {
      const mockSetReviewPayload = jest.fn()
      const { getByLabelText, getByText } = render(
        <Grid>
          <AddReviewComponent
            onValidationChange={jest.fn()}
            setReviewPayload={mockSetReviewPayload}
          />
        </Grid>
      )

      const noRadio = getByLabelText(REVIEW_SCREEN_NO_OPTION)
      fireEvent.click(noRadio)

      const textarea = getByLabelText(REVIEW_SCREEN_COMMENTS_LABEL)

      fireEvent.blur(textarea)
      fireEvent.change(textarea, { target: { value: 'ne' } })

      expect(getByText(MIN_LENGTH_COMMENTS_ERROR_MESSAGE)).toBeInTheDocument()
    })
  })

  // New Test Case: Ensure error message is shown when comment length is exactly the minimum required
  it('shows error message when comment length is exactly the minimum required', () => {
    const mockSetReviewPayload = jest.fn()
    const { getByLabelText, getByText } = render(
      <Grid>
        <AddReviewComponent
          onValidationChange={jest.fn()}
          setReviewPayload={mockSetReviewPayload}
        />
      </Grid>
    )

    const noRadio = getByLabelText(REVIEW_SCREEN_NO_OPTION)
    fireEvent.click(noRadio)

    const textarea = getByLabelText(REVIEW_SCREEN_COMMENTS_LABEL)
    fireEvent.change(textarea, { target: { value: 'abc' } }) // Assuming minimum length is 3
    fireEvent.blur(textarea)

    expect(getByText(MIN_LENGTH_COMMENTS_ERROR_MESSAGE)).toBeInTheDocument()
  })
})

describe('ViewReviewScreen Component', () => {
  describe('Initial Rendering of ViewReviewScreen', () => {
    afterEach(() => {
      jest.restoreAllMocks()
    })

    it('renders correctly with all inside elements when review data is present', () => {
      jest
        .spyOn(reviewsApiHook, 'useReviewsQuery')
        .mockImplementation(() => reviewsHookMockData[0] as never)

      const { getByText } = render(
        <Grid>
          <ViewReviewComponent />
        </Grid>
      )

      const requetedChangesTitle = getByText(REQUESTED_CHANGES_TITLE)
      expect(requetedChangesTitle).toBeInTheDocument()
    })

    it
