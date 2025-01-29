    it('should show error toast when API call fails', () => {
      jest.spyOn(reviewsApiHook, 'useReviewsQuery').mockImplementation(() => ({
        isError: true,
        error: {
          data: {
            Errors: {
              Error: [{
                Details: 'Custom error message'
              }]
            }
          }
        }
      } as never));

      const { getByText } = render(
        <Grid>
          <ViewReviewComponent />
        </Grid>
      );

      expect(getByText('Custom error message')).toBeInTheDocument();
    });
